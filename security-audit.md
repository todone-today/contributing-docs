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
