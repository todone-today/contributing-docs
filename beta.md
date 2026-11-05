# Beta testing

How todone's beta runs: who tests, what they try, how feedback comes back, and when it's done.

## Who

10–20 people, mixed on purpose: a few who plan their week in a calendar, a few who live in Notion or Obsidian, a small team of 3–5 who'll share a workspace, and at least three on phones only. Invite them to [todone.today](https://todone.today); anyone unsure can look around [the demo](https://todone.today/demo) first.

## What to try

Send testers this list and ask them to use todone for their real tasks for two weeks.

1. Make an account, take the tour, and delete the sample lists.
2. Add a week of real tasks; give them days, times and lengths with stickers.
3. Plan a day on the calendar; stretch a block to change how long it takes.
4. Dump ideas in Brainstorm and turn two into tasks.
5. Use Import with AI on a brain dump or meeting notes.
6. Make a team, invite someone, and work on one list together.
7. Turn on notifications on your phone and wait for a reminder.
8. Use it on your phone, and once with no connection.
9. Connect one integration you'd really use (Google Calendar, Notion, Obsidian or Discord).
10. Export your data.

## Feedback

- **In the app:** "Send feedback" (bug, idea or other); it's saved with the page it came from. Read it with `npm run feedback` in the [server](https://github.com/todone-today/server) repo (against production: `vercel env pull` the production settings first).
- **On GitHub:** [issues](https://github.com/todone-today/clients/issues/new/choose) with the bug and idea forms.
- **Talk to them:** a 15-minute call with five testers at the end of week one; ask what they stopped using and why.

Triage twice a week: label each piece of feedback bug or idea, fix what blocks people first, and reply to everyone who wrote.

## Questions to answer

- Do people come back on day 3 and day 10 without being reminded?
- Which of stickers, calendar, Brainstorm and Import with AI do they actually use?
- Where do they get stuck in the first five minutes?
- Do teams keep sharing a list after the first week?
- What do they want that isn't there?

## Done when

Two weeks have passed, nothing reported blocks someone from using todone daily, every bug is fixed or has an issue, and the answers above are written up here with what changes next.
