# Pricing and the cloud business

The recommendation behind the plans on [todone.today](https://todone.today/#pricing), and the numbers it rests on. Prices live in `clients/apps/web/src/lib/landing.ts`; change them there.

## The model in one line

todone stays free and open source; self-hosting is free forever; the hosted cloud is **freemium with generous limits**, and people pay for heavy use and for teams. Nobody pays to use todone the way most people use a todo list.

Why freemium rather than paid-from-day-one or free-unlimited:

- **Paid from day one** fights the open-source story (anyone can self-host the same thing) and kills word of mouth before there is any.
- **Free, unlimited** has no path to paying for servers, and invites abuse (scripts creating thousands of lists).
- **Generous freemium** keeps the everyday user free, which is the growth engine, and charges the people who get the most value: power users and teams. That's also where the real costs are.

## Plans

| Plan | India | International | Who it's for |
|---|---|---|---|
| Free | ₹0 | $0 | Everyday use: up to 10 lists, one team of up to 5, every feature, sync, offline, export |
| Plus | ₹99 / month · ₹499 / 6 months · ₹899 / year | $3 / month · $16 / 6 months · $29 / year | Power users: unlimited lists, teams up to 10, restorable daily backups, early mobile access |
| Team | ₹149 / person / month · ₹1,499 / person / year | $5 / person / month · $49 / person / year | Households and small teams: unlimited people and teams, roles, priority help |
| Founding member | ₹1,999 once | $59 once | Early supporters: 3 years of Plus, credits, and it funds the app-store accounts |
| Self-host | free | free | Anyone, every feature, GPL-3.0 |

How these were picked:

- **India first, honest international.** ₹99 a month sits with Indian productivity subscriptions (₹99–₹199) and is an easy UPI tap; $3 internationally sits just under Todoist Pro (about $4 a month billed yearly) and level with TickTick Premium (about $36 a year), fitting for a newer, smaller app. Check their current prices before launch. Not a straight currency conversion: purchasing power differs, and Razorpay shows the visitor's currency.
- **Launch prices (first term only).** 6 months: Plus ₹349 / $11, Team ₹549 / $18 a person. Yearly: Plus ₹599 / $19,
  Team ₹999 / $33 a person. They renew at the normal prices above, and the card always says so (India's 2023 dark
  patterns guidelines rule out hiding the renewal). The toggle's "save %" compares the first term with paying monthly
  for as long, rounded down: yearly 49%, 6 months 41%. Founding (₹1,999 for 3 years) still beats yearly at launch
  prices (₹599 + ₹899 + ₹899 = ₹2,397). Values live in `clients/apps/web/src/lib/landing.ts`.
- **Annual is the plan to sell.** ₹899 is ~25% off twelve months (₹1,188); six months (₹499) is ~16% off. Expect most paying users on annual: it cuts churn and the per-charge payment fee, and the six-month option catches people who won't commit to a year. Monthly exists so the first payment is small.
- **Team per person**, since cost and value both scale with people.

## No lifetime cloud

A one-off payment for unlimited hosted cloud forever is a promise to pay hosting costs forever from money that's already spent. It also tends to sell to exactly the heaviest users. So:

- **Founding member** is three years of Plus, paid once. It feels like a lifetime deal to supporters, has an end date, and funds the mobile developer accounts.
- If a true lifetime product is ever wanted, make it a **lifetime licence for the self-hosted app's extras** (say, a supporter build or paid support), never lifetime cloud.

## The economics

Per paying user, per year, at ₹899 (≈ $10.70):

| Cost | Estimate |
|---|---|
| Razorpay fees | ~2% domestic (UPI may be 0%), ~3% international, plus 18% GST on the fee: ~₹20–30 |
| GST on the sale | 18% if/when registered (turnover above ₹20 lakh, or exporting services): ~₹137 of ₹899 |
| Hosting (Vercel functions + Neon Postgres, shared) | Pennies per user at this scale; ~₹5–20 a year for a heavy user |
| Backups and bandwidth | Small; tasks are text |
| Support | The real cost: your time |

So each Plus subscriber nets roughly ₹700–850 a year. The fixed costs that matter are the hosting tiers once free tiers run out: Vercel Pro (about $20 a month) and Neon's paid plan (from about $19 a month; check current prices), about ₹3,300 a month together. **Around 50 annual Plus subscribers cover the hosting**; everything after that funds development.

The free tier's cost is small (text data, idle most of the time), so it can stay generous. Watch for the exception, scripted abuse, and handle it with rate limits rather than smaller limits.

Customer acquisition should cost close to nothing: open source, the demo, word of mouth, the changelog, and communities (Indian dev and productivity groups, Product Hunt, Hacker News on a real launch). Paid ads don't pay back at ₹899.

## Cloud limits and fairness

- **Free:** 10 lists, 1 team of 5, unlimited tasks within them, every integration. Enough for almost everyone.
- **Abuse:** rate-limit writes per account and per IP at the API, cap list size (already validated server-side), and block signup bursts. None of this should ever be noticed by a normal user.
- **Backups:** Plus and Team get daily backups restorable from the Account page; that's the paid feature with real cost, and it must ship before Plus does.
- **Teams:** the free team of 5 covers a household; bigger teams are where Team pricing starts.
- None of the limits exist in code yet. Enforce them when checkout launches, never retroactively without notice, and grandfather anyone already over them.

## What has to exist before charging

1. Razorpay checkout and webhooks on the server; a `plan` on each account; the limits above enforced.
2. Daily backups and restore.
3. Pricing, refunds and cancellation updated in the Terms and the Refund & Cancellation Policy (both currently say todone is free).
4. GST registration if required; invoices from Razorpay.
5. Set `PAY_URL` in `landing.ts`, and update `FUNDING.raisedUsd` as purchases come in. When it reaches $124, set `done: true` and the funding section steps aside.
