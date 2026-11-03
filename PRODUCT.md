# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

Repos by concern, Bitwarden-style (folders in this repo for now): `clients/apps/web` (Next.js App Router, deployed on Vercel; pushes to `develop` deploy previews, `main` production) and `contributing-docs`. The terminal app is retired (tag `tui-final`).

## Users

People and small teams who want a calm, fast todo list they can see and move around: sticky notes, a calendar, shared workspaces.

## Product Purpose

Lists of tasks as sticky notes with steps, due dates and times, estimates, priority and repeats; a day/week/month/year calendar; Brainstorm for loose ideas; teams. It connects third-party services (Google Calendar, Notion, Discord, a calendar feed, Markdown sync for Obsidian) and stores tasks in the Obsidian Tasks Markdown style.

## Positioning

Open source and self-hostable, with an optional hosted cloud. The cloud has a free tier and a paid tier; paid only starts past usage far beyond normal (working idea: more than about 10 lists). Pricing is not decided; do not state prices or limits as fact.

## Operating Context

- Web: browser on desktop and phone, installable, works offline.
- Integrations live on the server side because OAuth needs a server.

## Capabilities and Constraints

- Shipped (web): accounts, teams and invitations, lists and sticky notes with drag and drop, stickers, repeats, calendar views and sharing, Brainstorm, onboarding tour, integrations, push reminders, offline, export, account deletion, feedback.
- Undecided: final product name, pricing, list limits.

## Brand Commitments

- Working name "todone"; a final name comes later, so keep the name easy to swap (one place).
- Visual references the user chose: https://zagged.com/ and https://two7square.com/about

## Evidence on Hand

- No customers, testimonials, user counts or press exist. Do not invent them.

## Product Principles

1. Direct manipulation: drag, drop and click before forms and menus.
2. Your data stays yours: open source, self-hostable, Markdown-compatible.
3. Fair cloud: free covers normal use; paying is for unusually heavy use.
4. Calm and fast over feature-heavy.
