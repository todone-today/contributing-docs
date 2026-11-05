# Contributing to todone

todone ([todone.today](https://todone.today)) is split into repos by concern, like [Bitwarden](https://github.com/bitwarden):

| Repo | What it is |
|---|---|
| [server](https://github.com/todone-today/server) | The API: accounts and teams, lists, integrations, reminders, migrations, Docker Compose and Drone |
| [clients](https://github.com/todone-today/clients) | The apps people use; today `apps/web`, the Next.js web app |
| [sdk](https://github.com/todone-today/sdk) | `@todone/sdk`, the logic both share |
| contributing-docs | This: setup, architecture, design, product and legal notes |

## How it fits together

```
browser ──> clients/apps/web (Next.js) ──/api/*──> server (Hono + Better Auth) ──> Postgres
                    │                                   │
                    └────────── @todone/sdk ────────────┘
```

- **server** owns all data and every rule about it: who may see or change what (`src/lib/access.ts`), validation, integrations and their secrets. Its routes are in `src/app.ts`; each calls a function in `src/actions/`.
- **clients/apps/web** is the UI. It never touches the database. Its `next.config.ts` forwards `/api/*` to the server, so the browser sees one address: the session cookie, OAuth callbacks and the calendar and Markdown links all belong to the web app. `src/lib/api.ts` is the only way it calls the server.
- **sdk** is plain TypeScript with no runtime dependencies: the task model and its Obsidian Tasks Markdown format, calendar maths, CSV export, sync planning and the API's response types (`src/api.ts`). Server and clients install it from GitHub at a tag (`github:todone-today/sdk#v0.1.0`), which builds it to JavaScript; they import it as `@todone/sdk/<module>`.

## Run it locally

Clone the repos side by side:

```sh
git clone https://github.com/todone-today/server
git clone https://github.com/todone-today/clients
```

```sh
cd server
npm install
npm run local        # Postgres in Node (PGlite, data in .pglite), migrations, API on :4000
```

```sh
cd clients/apps/web
npm install
npm run dev          # http://localhost:3000, forwarding /api to SERVER_URL (default http://localhost:4000)
```

The server reads `.env.local` (see its `.env.example`). Set `BETTER_AUTH_URL` to the web app's address (`http://localhost:3000`) and `BETTER_AUTH_SECRET` to a long random string.

With Docker instead: in the server repo, `docker compose up --build`, then open http://localhost:3000.

To change the sdk and try it before releasing, `npm install ../sdk` in the server or web app, then put the tag back once the sdk is tagged.

## Checks

```sh
sdk:             npm run typecheck && npm test && npm run build
server:          npm run typecheck && npm test
clients/apps/web: npm run lint && npm test && npm run build
```

Each repo's `.github/workflows/ci.yml` runs its own; the server's `.drone.yml` runs its checks on the Mac mini.

## Deploying

- **Vercel:** two projects. `todone` is the web app (clients repo, root `apps/web`, `SERVER_URL` = the server's address), served at https://todone.today (www redirects to it). `todone-server` is the server repo: `npm run vercel-build` bundles it into one function, and it holds the database and the auth, push and cron settings. Its `BETTER_AUTH_URL` is https://todone.today, and `BETTER_AUTH_TRUSTED_ORIGINS` lists the other addresses it serves (todone-flame.vercel.app, todone-develop.vercel.app). With a `VERCEL_TOKEN` secret in each repo, a push to `main` deploys it (the server migrates the database first).
- **Mac mini:** a push to the server repo's `main` makes Drone rebuild and restart its Docker Compose project on the host (Postgres, server, web built from the clients repo, reminders, and a Cloudflare Tunnel with `CLOUDFLARE_TUNNEL_TOKEN`). Mark the repo **Trusted** in Drone and add the settings as Drone secrets.

## More

- [DESIGN.md](https://github.com/todone-today/clients/blob/main/DESIGN.md) and [PRODUCT.md](https://github.com/todone-today/clients/blob/main/PRODUCT.md): the visual system and the product, kept next to the UI in the clients repo
- [pricing.md](pricing.md): plans, prices and the cloud business
- [legal.md](legal.md): the policies and the DPDP Act review
- [beta.md](beta.md): how the beta runs and how feedback comes back

The project began as a terminal app; that history lives in the archived [1stdevfriend/todone](https://github.com/1stdevfriend/todone) (tag `tui-final`).
