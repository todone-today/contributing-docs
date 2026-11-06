# Running the API on the VPS

Production since 6 November 2026: the website stays on Vercel (todone.today); the API and Postgres 17 run on a
Contabo VPS with Docker Compose, at https://api.todone.today. The website forwards `/api/*` there (`SERVER_URL`) and
adds a secret header (`PROXY_SECRET`); Caddy refuses anything without it, so the API can't be reached directly and
the client IPs it receives are real.

| Where | What |
|---|---|
| `/root/todone` | clone of `todone-today/server`; `.env` (root only) holds every secret |
| `docker compose -p todone` | `db`, `server`, `reminders` (calls the cron every 10 min), `caddy` (443 only) |
| `/root/todone-backups` | nightly `pg_dump` at 03:17 server time, 14 days kept (`/etc/cron.d/todone-backup`) |

Port 80 belongs to another app on this machine, so Caddy gets certificates by TLS-ALPN on 443 alone.

## Deploy a new server version

```sh
ssh root@158.220.113.177
cd /root/todone && git pull && docker compose -p todone --profile api up -d --build db server reminders caddy
```

Migrations run when the server starts.

## Restore a backup

```sh
cd /root/todone
docker compose -p todone exec -T db pg_restore -U todone -d todone --clean --if-exists < /root/todone-backups/todone-YYYY-MM-DD.dump
docker compose -p todone restart server
```

## Change a secret

Edit `/root/todone/.env`, then `docker compose -p todone up -d server` (or `caddy` for `PROXY_SECRET`, which must
also be changed on the Vercel `todone` project and redeployed).

## Roll back to Vercel + Neon

Set `SERVER_URL` on the Vercel `todone` project back to the old server's address and redeploy. Neon still has the data
as of the move, not anything written since; copy newer data back with `pg_dump --data-only` first.
