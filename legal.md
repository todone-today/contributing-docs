# Legal and compliance

Where each policy lives and why the app is shaped the way it is. The pages are drafts written to match what the code does; have a lawyer review them before relying on them.

| Policy | Page | Source |
|---|---|---|
| Privacy Policy (also the DPDP notice) | `/privacy` | `clients/apps/web/src/app/(legal)/privacy/page.tsx` |
| Terms & Conditions | `/terms` | `clients/apps/web/src/app/(legal)/terms/page.tsx` |
| Cookie Policy | `/cookies` | `clients/apps/web/src/app/(legal)/cookies/page.tsx` |
| Refund & Cancellation Policy | `/refunds` | `clients/apps/web/src/app/(legal)/refunds/page.tsx` |
| Acceptable Use Policy | `/acceptable-use` | `clients/apps/web/src/app/(legal)/acceptable-use/page.tsx` |

The operator, contact address and "last updated" date are in `clients/apps/web/src/lib/brand.ts`. Change the date there whenever a policy changes, and update the pages whenever the app starts collecting something new or uses a new provider.

## Licence

todone is GPL-3.0 (`LICENSE`). That covers the terminal app and the web app's code.

One gap to decide on: the GPL's share-alike rule is triggered by *distributing* the software. Someone who runs a modified todone web app as a hosted service doesn't distribute it, so they don't have to publish their changes. The AGPL-3.0 closes that gap (it treats letting people use it over a network as distribution). Switch the web app to AGPL-3.0 if hosted forks should share their changes; keep GPL-3.0 if that's fine. As the only author, you can relicense.

## DPDP Act, 2023 review

India's Digital Personal Data Protection Act and the DPDP Rules, 2025. The operator is the Data Fiduciary; Vercel, Neon, GitHub, Google, Notion, Discord and push services are processors.

| Requirement | Status | How |
|---|---|---|
| Notice before consent: what data, which purposes, how to withdraw, how to complain (s.5) | Done | `/privacy`, linked from sign-up |
| Free, specific, informed consent by a clear affirmative action (s.6) | Done | Required checkbox on email sign-up; notice under "Continue with GitHub" |
| Withdraw consent as easily as it was given (s.6(4)) | Done | Account → Delete account |
| Use data only for the stated purposes (s.6) | Done | No analytics, ads or AI training; integrations only when switched on |
| Right to a summary of data and who it was shared with (s.11) | Done | Account → Export everything; summary on request by email |
| Right to correction and erasure (s.12) | Done | Edit tasks and name in the app; delete anything; email change by request |
| Right to grievance redressal (s.13) | Done | Grievance contact and timelines in `/privacy` |
| Right to nominate (s.14) | Done | By email, described in `/privacy` |
| Erase data when consent is withdrawn (s.8(7)) | Done | Account deletion removes the user, lists, sessions, tokens, push addresses, feedback and solo teams (`clients/apps/web/src/lib/auth.ts`) |
| Reasonable security safeguards (s.8(5)) | Done | HTTPS, hashed passwords, per-workspace authorisation on every server action, secrets in env vars |
| Children: verifiable parental consent, or don't serve them (s.9) | Done | 18+ only, stated in Terms and confirmed at sign-up |
| Breach notification to people and the Board (s.8(6)) | Process | Committed to in `/privacy`; no automated alerting. If a breach happens, email affected users and report to the Data Protection Board within 72 hours |
| Cross-border transfer (s.16) | Done | Providers are outside India, which is allowed unless the government restricts a country; disclosed in `/privacy` |
| Proof of consent (s.6(10)) | Partial | A user's `createdAt` plus the policy date shows which version they agreed to. If you need a per-user record, store the policy version at sign-up |
| Significant Data Fiduciary duties (DPO, audits, DPIA) | Not applicable | Only if the government notifies todone as one |

Things to keep true as the app grows: add any new data or provider to `/privacy` before shipping it, keep account deletion removing everything new tables hold (cascade from `"user"` or clean up in `afterDelete`), and keep the contact address answered.
