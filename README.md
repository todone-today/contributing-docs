# Contributing to todone

## How it fits together

```
browser ──> clients/apps/web (Next.js) ──/api/*──> server (Hono + Better Auth) ──> Postgres
                    │                                   │
                    └────────── @todone/sdk ────────────┘
```

- **server/** owns all data and every rule about it: who may see or change what (`src/lib/access.ts`), validation, integrations and their secrets. Its routes are in `src/app.ts`; each calls a function in `src/actions/`.
- **clients/apps/web/** is the UI. It never touches the database. Its `next.config.ts` forwards `/api/*` to the server, so the browser sees one address: the session cookie, OAuth callbacks and the calendar and Markdown links all belong to the web app. `src/lib/api.ts` is the only way it calls the server.
- **sdk/** is plain TypeScript with no dependencies: the task model and its Obsidian Tasks Markdown format, calendar maths, CSV export, sync planning, and the API's response types (`src/api.ts`). Both sides import it as `@todone/sdk/<module>`.

Node 24 runs the server and the tests as TypeScript directly; there's no build step outside the web app.

## Run it locally

```sh
cd server
npm install
npm run local        # Postgres in Node (PGlite, data in server/.pglite), migrations, API on :4000
```

```sh
cd clients/apps/web
npm install
npm run dev          # http://localhost:3000, forwarding /api to SERVER_URL (default http://localhost:4000)
```

The server reads `server/.env.local` (see `server/.env.example`). Set `BETTER_AUTH_URL` to the web app's address (`http://localhost:3000`) and `BETTER_AUTH_SECRET` to a long random string.

With Docker instead: `docker compose -f server/docker-compose.yml up --build`, then open http://localhost:3000.

## Checks

```sh
cd sdk && npm test && npm run typecheck
cd server && npm test && npm run typecheck
cd clients/apps/web && npm run lint && npm test && npm run build
```

CI (`.github/workflows/ci.yml`) and Drone (`.drone.yml`) run the same.

## Deploying

- **Mac mini:** a push to `main` makes Drone rebuild and restart `server/docker-compose.yml` on the host (Postgres, server, web, reminders, and a Cloudflare Tunnel with `CLOUDFLARE_TUNNEL_TOKEN`). Mark the repo **Trusted** in Drone and add the settings as Drone secrets.
- **Vercel:** the web app is one project (root `clients/apps/web`) with `SERVER_URL` pointing at the server.

## More

- [DESIGN.md](DESIGN.md): the visual system
- [PRODUCT.md](PRODUCT.md): who it's for and what it does
- [legal.md](legal.md): the policies and the DPDP Act review
