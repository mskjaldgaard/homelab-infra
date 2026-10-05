# homelab-infra

Wrapper repo for running [Homelable](https://github.com/mskjaldgaard/homelable) on this machine.

## Layout

- `homelable/` — git submodule, the Homelable app itself (backend, frontend, MCP server, docker-compose.yml).
- `homelable-data/` — local backup destination, outside the Docker volume. Tracked in git, a sibling folder to `homelable/`, written to by the `backup` service below.

## Running the stack

```bash
cd homelable
docker compose up -d
```

See `homelable/README.md` and `homelable/INSTALLATION.md` for the app's own configuration (`.env`, auth, network scanner, MCP server).

## Backups

`homelable/docker-compose.yml` defines a `backup` service (built from `homelable/backup/`) that runs alongside `backend`, `mcp` and `frontend`:

- Takes a consistent snapshot of the live SQLite DB with `sqlite3 .backup` — safe to run while the backend is writing to it, no need to stop the stack.
- Copies `uploads/` too, if present.
- Runs on a cron schedule **inside the container** (`homelable/backup/crontab`, daily at 03:00 container time) — no host-side cron/launchd job, so it works the same on any machine running the stack.
- Writes to `../homelable-data` (i.e. this repo's `homelable-data/`) via a bind mount; override the destination with `BACKUP_DIR` and retention (default 14 days) with `BACKUP_RETENTION_DAYS` in `homelable/.env`.

Trigger a backup manually:

```bash
docker exec homelable-backup-1 /usr/local/bin/backup.sh
```