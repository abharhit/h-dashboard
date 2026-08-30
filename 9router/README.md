# 9router backup

This folder holds the 9router `data.sqlite` that the `maity` workflow automatically
restores onto each fresh runner, so you never have to re-do the dashboard setup
(provider = opencode, model = hy3, combo = code, copied API key).

## File committed here

- `data.sqlite` -> copied to `~/.9router/db/data.sqlite`

Only `data.sqlite` is needed. The `jwt-secret` is NOT backed up on purpose:
the workflow sets `INITIAL_PASSWORD` (via `RUNNER_PASSWORD`) so dashboard login
works with a freshly generated secret. (Verified: login succeeds with only the db
restored.)

## How to refresh the backup from this server

9router must be running here (it is). Flush the WAL, then copy:

```bash
sqlite3 ~/.9router/db/data.sqlite "PRAGMA wal_checkpoint(TRUNCATE);"
cp ~/.9router/db/data.sqlite /path/to/my-tail/9router/data.sqlite
cd /path/to/my-tail
git add 9router/data.sqlite
git commit -m "Refresh 9router db backup"
git push origin main
```

Next workflow run will pick up the new state automatically.

## What the workflow does

In step 12 it copies `9router/data.sqlite` -> `~/.9router/db/data.sqlite`, then
starts 9router hidden in the background (tray mode) on port 20128:

```bash
nohup 9router --tray --skip-update -p 20128 > /tmp/9router.log 2>&1 &
```

> Note: `data.sqlite` is committed to this repo on purpose (free models only,
> no paid keys). Do NOT put real paid provider keys in here.
