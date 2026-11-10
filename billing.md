# Billing: how payments work, and the GST checklist

Paid plans run through Razorpay. Code: `server/src/actions/billing.ts` (checkout, webhooks, refunds),
`server/src/lib/plans.ts` (prices, limits, who gets what) and `clients/apps/web/src/app/app/Billing.tsx` (the Plan page).
Prices exist twice: `lib/plans.ts` charges, `clients/apps/web/src/lib/landing.ts` shows. Change both together.

## How it works

- **Plus and Team are Razorpay subscriptions.** Each one starts at a future date (`start_at`) with an upfront amount paid
  now (`addons`):
  - the first purchase pays its first term upfront (at the launch price where there is one) and renews at the regular
    price from then on;
  - an upgrade (Plus → Team, more seats) pays the difference for the days already paid for and renews at the new price
    when the old one would have;
  - a downgrade or a change of billing period starts at the next renewal, with nothing paid now.

  The old subscription is cancelled at its cycle end. Razorpay won't update UPI subscriptions in place, so this works the
  same for UPI, cards and netbanking.
- **Founding member** is one Razorpay order: three years of Plus, no renewal.
- **What someone gets** is worked out on every request from `billing_sub.paid_until`, so there's no expiry job. While
  Razorpay retries a failed renewal there are 3 days' grace.
- **Limits only stop new things:** an 11th list, a team, an invite, a share link. Nothing is ever locked or deleted.
- **Grandfathered:** accounts created before `BILLING_START` have no limits, for good.
- **Self-hosting:** without the Razorpay keys and `BILLING_START`, billing is off and there are no limits.
- **Refunds:** in the app, within 7 days of any payment. The whole payment is refunded and that plan ends. Anything else
  is refunded from the Razorpay dashboard, and the `refund.processed` webhook records it.
- **Receipts:** every payment gets a number (`TD-<year>-<serial>`) and a printable page at `/app/receipt/<payment id>`.
  Payment rows are kept after an account is deleted (tax records).

## Going live

1. In the Razorpay dashboard, switch to Live and make new keys. The test keys were shared in a chat, so don't reuse that
   secret anywhere: regenerate the test secret too.
2. Webhooks → add `https://todone.today/api/billing/webhook`, choose a secret, and tick:
   - `subscription.authenticated`, `subscription.activated`, `subscription.charged`, `subscription.pending`,
     `subscription.halted`, `subscription.cancelled`, `subscription.completed`;
   - `order.paid`;
   - `refund.processed`.
3. On the VPS `.env`, set:
   - `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` and `RAZORPAY_WEBHOOK_SECRET`;
   - `BILLING_START`, the moment limits begin (an ISO time; everyone who signed up before it is grandfathered);
   - optionally `BILLING_NAME`, `BILLING_ADDRESS` and `BILLING_GSTIN`.
4. Deploy the server, then use the dashboard's "Send test webhook" and check the server log.
5. Make one real ₹99 Plus monthly payment, then refund it from the app. You've then seen the whole loop.

Razorpay plans (`billing_plan`) are made on first use. Test and live have separate plan ids, and the table is keyed by
price, so switching modes or prices just makes new ones. If you switch an existing database from test to live, clear
`billing_plan` first.

## GST and invoicing (a checklist for you or your CA, not legal advice)

- **Registration.** Services need GST registration once turnover passes ₹20 lakh a year (₹10 lakh in special-category
  states). Below that it's optional. Selling only in rupees to people in India, inter-state services below the threshold
  don't force registration. Selling to people abroad (exports, OIDAR) changes this, so ask before taking foreign cards.
- **Prices include tax.** The site and the app say "prices include taxes", so registering doesn't change what people
  pay. GST then comes out of the price: 18%, about ₹137 of ₹899.
- **Invoices.** While unregistered, a receipt (what todone makes now) is enough; don't charge or show GST.
  Once registered:
  - set `BILLING_GSTIN`, `BILLING_NAME` and `BILLING_ADDRESS`;
  - receipts become "Tax invoice" with the 18% shown as included;
  - the numbering is consecutive within a financial year, as invoices must be. It's per calendar year now (`TD-2026-…`),
    so check with the CA whether you want it to restart in April. The serial is a Postgres sequence.
  - For B2B buyers who want input credit, the invoice also needs their GSTIN and the place of supply. Add a field to the
    Plan page if businesses start asking.
- **e-invoicing (IRN)** only applies above ₹5 crore turnover.
- **Razorpay's own fees** carry 18% GST, which you can claim back once registered.
- **Records.** Keep payment records for 6 years after the end of the year they relate to (GST) and 8 years under the
  Companies Act if you incorporate. `billing_payment` keeps them even when an account is deleted. Back up the database
  (see vps.md).
