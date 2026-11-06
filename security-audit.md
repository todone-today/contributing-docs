# Security audit — 6 November 2026

A pass over the server (Hono + Better Auth + Postgres) and the web client, ahead of beta. Found problems are fixed in
the same change; the rest is what was checked and why it holds. Report vulnerabilities privately (see SECURITY.md in
the .github repo), not in issues.

## Fixed

| Issue | Fix |
|---|---|
| No rate limiting: sign-in, sign-up, reset and feedback could be hammered | Better Auth's limiter, stored in Postgres (sign-in 10/min, sign-up 10/h, reset and verification emails 5/h, change password and delete account 10/h, the rest 100/min). Our API: a fixed-window `rate_limit` table per user or IP: feedback 10/h, token routes (sync, calendar, shared) 60/min, demo 30/min, writes 120/min, reads 600/min. Over the limit is `429` with `retry-after`. Works across serverless instances because it lives in the database. |
| Push subscriptions took any URL, so the server could be made to POST to internal hosts (SSRF) | Endpoints must be https on a known push service (FCM, Mozilla, Apple, Windows), checked on subscribe and again before sending. |
| No security headers | Server: Hono `secureHeaders()`. Web: `X-Frame-Options: DENY`, CSP `frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'`, `nosniff`, strict referrer policy, HSTS (2 years), camera/microphone/location/payment off. |
| Accounts with unconfirmed emails lived forever | Email confirmation; unconfirmed accounts are deleted after 7 days. |

## Checked, holds

- **Authorisation.** Every list, team and integration route goes through `who()` and filters by the caller's user or
  organisation membership in SQL; a list id from another account returns 404, not its data.
- **SQL injection.** All queries use `pg` parameters; no string-built SQL with user input.
- **CSRF.** Cookies are `SameSite=Lax` and Better Auth checks `Origin` against trusted origins; the API only accepts JSON
  bodies.
- **XSS.** React escapes everything; the one `dangerouslySetInnerHTML` is the static theme script. Emails escape every
  inserted value.
- **Token routes** (calendar feed, Markdown sync, shared views) use random tokens, return only that token's data, and
  are rate limited.
- **Cron** needs `CRON_SECRET`.
- **Discord webhooks** must be `discord.com`/`discordapp.com` webhook URLs.
- **Request size** capped by `bodyLimit`.
- **Secrets** live only in env vars (Vercel encrypted, `.env.local` git-ignored); none are in the repos or their history.
- **Dependencies.** `npm audit --omit=dev`: 0 known vulnerabilities in server and clients.

## Follow-ups

- Rotate any key that has been shared outside the env store (the Resend key was pasted into a chat).
- ~~Production has no cron scheduler.~~ Since the move to the VPS, the `reminders` service calls the cron every 10
  minutes.
- A full Content-Security-Policy with `script-src` needs nonces from Next; the current CSP covers framing, forms and
  plugins only.
- Re-run this audit before payments go live (Razorpay webhooks need signature checks).

## End-to-end encryption (zero knowledge), 6 November 2026

Lists are encrypted in the browser; the server stores ciphertext it can't open. Code: `sdk/src/crypto.ts` (tested in
`crypto.test.ts`), `clients/apps/web/src/lib/vault.ts`, `server/src/actions/keys.ts`. WebCrypto only.

- **Password → keys.** PBKDF2-SHA256, 600,000 iterations, a random 16-byte salt per account → master key. HKDF gives the
  key that wraps the account key; PBKDF2(master key, password, 1) gives the **auth hash**, which is what the browser
  sends as "the password". The server hashes it again (scrypt, Better Auth). The password itself is never sent.
- **Account key**: random AES-256-GCM, stored wrapped by the password key and by the recovery key (32 random bytes, shown
  once as 52 base32 characters), plus the recovery key itself encrypted by the account key so Account can show it again.
- **Lists**: `v1.<iv>.<ciphertext>` of `{name, tasks}`, with the list id as additional data so blobs can't be swapped
  between lists. Personal lists use the account key; team lists the team key.
- **Teams**: each member has an RSA-OAEP-2048 key pair (private key wrapped by the account key). A team's AES key is
  wrapped to each member's public key. Only someone holding it can give it to a newcomer ("Confirm"), and existing copies
  are never overwritten.
- **On the device**: unlocked keys are kept in IndexedDB as non-extractable `CryptoKey`s until sign-out; the offline
  copy of lists in `localStorage` is encrypted with the account key.
- **Changing password** re-wraps the account key in one transaction with the new hash; other sessions are signed out.
  **Forgot password** needs the emailed link *and* the recovery key; without the key, "start fresh" wipes the
  encrypted lists (nobody can open them) and keeps the login.
- **prelogin** gives unknown emails a stable fake salt (HMAC of the email), so it doesn't reveal who has an account.
  It's rate limited, like the recovery endpoints.

What the server can still see: email, name, team memberships, how many lists, their sizes and timestamps, and lists a
person switched off encryption for (only those reach integrations, the calendar feed and shared links).

Known limits:
- Removing a member doesn't rotate the team key (as in Bitwarden). Server access checks still keep them out.
- Confirming a member trusts the public key the server returns; the fingerprint (Account) is how to check it.
- A compromised server could ship altered JavaScript; that's true of every web app doing end-to-end encryption.
- Reminders run in the open app (any tab), since the server can't see due dates.
