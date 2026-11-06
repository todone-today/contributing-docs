# Running the API on the VPS

Production since 6 November 2026: the website stays on Vercel (todone.today); the API and Postgres 17 run on a
Contabo VPS with Docker Compose, at https://api.todone.today. The website forwards `/api/*` there (`SERVER_URL`) and
adds a secret header (`PROXY_SECRET`); Caddy refuses anything without it, so the API can't be reached directly and
the client IPs it receives are real.

| Where | What |
|---|---|
| `/root/todone` | clone of `todone-today/server`; `.env` (root only) holds every secret |
| `docker compose -p todone` | `db`, `server`, `reminders` (calls the cron every 10 min), `caddy` (443 only) |
| `/root/todone-backups` | nightly `pg_dump` at 03:17 server time, 14 days kept (`scripts/backup.sh`, run by `/etc/cron.d/todone-backup`, log in `/var/log/todone-backup.log`) |
| R2 `todone-backups` | the same dumps encrypted with `age` to the owner's key (`/root/todone/backup.age-recipient`): `daily/` 15 days, `monthly/` a year. rclone remote `r2` in `/root/.config/rclone/rclone.conf` |

Every run checks in with Sentry Crons (`todone-backup`), which emails when a night fails or doesn't happen. On
Sundays the script also restores the dump into a throwaway Postgres and compares row counts; run that any time with
`/root/todone/scripts/backup.sh --test`.

Monitoring: UptimeRobot checks `https://todone.today` and `https://todone.today/api/health` (must contain
`"db":true`) every 5 minutes. Errors from the server and the website go to Sentry (`todone-server`, `todone-web`, EU
region) with no request, user or IP data.

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

## Restore from R2 (the VPS is gone)

On any machine with `rclone` (same `r2` remote) and the private key:

```sh
rclone copy r2:todone-backups/daily/todone-YYYY-MM-DD.dump.age .
age -d -i todone-backup.key -o todone.dump todone-YYYY-MM-DD.dump.age
pg_restore -d "$DATABASE_URL" --clean --if-exists --no-owner todone.dump
```

## Change a secret

Edit `/root/todone/.env`, then `docker compose -p todone up -d server` (or `caddy` for `PROXY_SECRET`, which must
also be changed on the Vercel `todone` project and redeployed).

## Roll back to Vercel + Neon

Set `SERVER_URL` on the Vercel `todone` project back to the old server's address and redeploy. Neon still has the data
as of the move, not anything written since; copy newer data back with `pg_dump --data-only` first.
