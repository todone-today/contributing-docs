# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

Next.js (App Router) in `web/`, deployed on Vercel. Pushes to `develop` deploy previews; `main` deploys production. The existing CLI (`todone`, one POSIX sh file) stays in the repo root.

## Users

People who want a calm, fast todo list. The web app is the primary product and the first door; the terminal app is the second door, for developers who want the same lists where they already work.

## Product Purpose

Lists of tasks with subtasks, due dates, time estimates, a calendar of what is due and a focus timer. The web app holds the lists, connects third-party services (OAuth integrations such as Google Calendar and Notion) and syncs with the terminal app, whose tasks are plain Markdown checkboxes.

## Positioning

Open source and self-hostable, with an optional hosted cloud. The cloud has a free tier and a paid tier; paid only starts past usage far beyond normal (working idea: more than about 10 lists). Pricing is not decided; do not state prices or limits as fact.

## Operating Context

- Web: browser on desktop and phone.
- Terminal: the `todone` CLI with a full-screen TUI; lists are `.md` files per folder or in `~/.todone`, in the Obsidian Tasks field style (📅 due, ⏱ estimate, ⏳ time spent).
- Integrations live on the web side because OAuth needs a server; the CLI syncs with the web app.

## Capabilities and Constraints

- Shipped (CLI): lists per folder, home screen of lists, tabs, subtasks, due dates, estimates, month calendar, focus timer, move tasks between lists, rename/delete lists, undo, Apple Reminders sync (untested against real Reminders).
- Web app: being built; first version is lists and tasks.
- Undecided: final product name, pricing, list limits, which integrations ship first.

## Brand Commitments

- Working name "todone"; a final name comes later, so keep the name easy to swap (one place).
- Visual references the user chose: https://zagged.com/ and https://two7square.com/about

## Evidence on Hand

- `demo/demo.gif`, `demo/demo.mp4`, `demo/home.png`, `demo/list.png`, `demo/calendar.png`, `demo/focus.png`: real CLI screenshots.
- No customers, testimonials, user counts or press exist. Do not invent them.

## Product Principles

1. Web first, terminal second: every feature works in the browser; the terminal mirrors it.
2. Your data stays yours: open source, self-hostable, Markdown-compatible.
3. Fair cloud: free covers normal use; paying is for unusually heavy use.
4. Calm and fast over feature-heavy.
