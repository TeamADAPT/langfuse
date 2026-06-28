# Langfuse Production Systemd Install Plan

## 2026-06-28 06:38:57 — Codex

## Objective
Install Langfuse production single-node on this host for `https://langfuse.adaptdev.ai`, managed by systemd, without Docker or Python virtual environments.

## Current Facts
- Host OS: Ubuntu 24.04.4 LTS.
- Passwordless sudo works.
- Repository branch was switched from `main` to `working`.
- `ops/` and `plans/` did not exist; this plan created the required scaffolding.
- Repo version: Langfuse `3.201.1`.
- Repo requires Node `24`; host currently has Node `22.23.1`.
- `pnpm` resolves to repo-pinned `11.4.0` through Corepack.
- Postgres 16 is active on `127.0.0.1:5432`.
- ClickHouse is active on `127.0.0.1:8123` and `127.0.0.1:9000`.
- Redis is listening on `127.0.0.1:6379`.
- Port `3000` is already occupied; Langfuse web will use `127.0.0.1:3030`.
- No Langfuse systemd units exist yet.
- No nginx or Caddy install was found.
- No certificate/key for `langfuse.adaptdev.ai` was found under `/etc`.
- `/adapt/secrets/.env` exists but currently only exposes generic API variables when inspected by key name.
- `/adapt/secrets/db.env` exists and contains shared infrastructure DB variables, including ClickHouse values, when inspected by key name.

## External Inputs Still Needed
These cannot be invented safely:
- Cloudflare R2 bucket name.
- Cloudflare R2 account ID or full S3 endpoint URL.
- Cloudflare R2 access key ID.
- Cloudflare R2 secret access key.
- Origin TLS certificate path and private key path for `langfuse.adaptdev.ai`, unless approval is given to install Caddy and provision TLS through another route.

Defaults I will use unless rejected:
- Initial owner email: `admin@adaptdev.ai`.
- Initial owner password: generated locally and written only to `/adapt/secrets/.env`.
- Initial org: `ADAPT`.
- Initial project: `ADAPT Langfuse`.
- Initial Langfuse API keys: generated locally and written only to `/adapt/secrets/.env`.
- Internal web bind: `127.0.0.1:3030`.
- Systemd service user/group: `x:x`, matching existing local production services.
- Reverse proxy: Caddy with manual TLS certificate paths.

## Implementation Steps After Approval
1. Prepare runtime secret files.
   - Back up `/adapt/secrets/.env` and `/adapt/secrets/db.env`.
   - Append a clearly delimited Langfuse block to `/adapt/secrets/.env`.
   - Append Langfuse Postgres, ClickHouse, and Redis connection variables to `/adapt/secrets/db.env`.
   - Generate `NEXTAUTH_SECRET`, `SALT`, `ENCRYPTION_KEY`, admin password, init IDs, and init API keys locally.
   - Set `NEXTAUTH_URL=https://langfuse.adaptdev.ai`.
   - Set `AUTH_DISABLE_SIGNUP=true` and `NEXT_PUBLIC_SIGN_UP_DISABLED=true`.
   - Configure R2 for event and media storage with S3-compatible variables:
     - `LANGFUSE_S3_EVENT_UPLOAD_*`
     - `LANGFUSE_S3_MEDIA_UPLOAD_*`
     - prefixes `events/` and `media/`
     - `LANGFUSE_ENABLE_BLOB_STORAGE_FILE_LOG=true`

2. Prepare system dependencies.
   - Install or upgrade system-wide Node 24.
   - Enable Corepack and activate `pnpm@11.4.0`.
   - Install/build `/usr/local/bin/migrate` with the ClickHouse driver, matching the repo Dockerfile intent.
   - Keep all execution system-wide; no Docker and no venv.

3. Prepare databases.
   - Create dedicated Postgres role/database: `langfuse`.
   - Create dedicated ClickHouse database/user: `langfuse`.
   - Use single-node ClickHouse config: `CLICKHOUSE_CLUSTER_ENABLED=false`.
   - Validate Redis connectivity and set `REDIS_HOST=127.0.0.1`, `REDIS_PORT=6379`.
   - If Redis auth is required, store it only in `/adapt/secrets/db.env`.

4. Build Langfuse.
   - Install dependencies with `pnpm install --frozen-lockfile`.
   - Build using Node 24 and production env loaded from `/adapt/secrets/.env` plus `/adapt/secrets/db.env`.
   - Use the repo's standalone Next.js output: `web/.next/standalone/server.js`.
   - Build worker output under `worker/dist`.

5. Run migrations.
   - Apply Prisma/Postgres migrations against the dedicated Postgres database.
   - Apply ClickHouse migrations from `packages/shared/clickhouse/scripts/up.sh`.
   - Keep migrations as an explicit `langfuse-migrate.service` oneshot dependency rather than hiding them in web startup.

6. Install systemd units.
   - `langfuse-migrate.service`: oneshot migration unit.
   - `langfuse-web.service`: Next standalone server on `127.0.0.1:3030`.
   - `langfuse-worker.service`: worker from `worker/dist/index.js`.
   - `langfuse.target`: grouped lifecycle target.
   - Enable restart policies and journal logging.

7. Install reverse proxy.
   - Install Caddy if absent.
   - Configure `langfuse.adaptdev.ai` to reverse proxy to `127.0.0.1:3030`.
   - Use manually supplied origin certificate and private key paths.

8. Verify live receipts.
   - `systemctl status langfuse.target langfuse-web langfuse-worker`.
   - `journalctl -u langfuse-web -u langfuse-worker` startup excerpts.
   - `curl http://127.0.0.1:3030/api/public/health?failIfDatabaseUnavailable=true`.
   - `curl https://langfuse.adaptdev.ai/api/public/health?failIfDatabaseUnavailable=true`.
   - Confirm signup is disabled via sign-in page/API behavior.
   - Use provisioned project API keys to ingest a live trace.
   - Confirm ingestion through HTTP response, worker logs, Postgres state, and ClickHouse rows.

## ClickHouse Rules Checked
- No Langfuse ClickHouse schema, query, or insert strategy changes are planned.
- Existing repo migrations remain the source of truth.
- Per general ClickHouse configuration guidance, this single-node install will set `CLICKHOUSE_CLUSTER_ENABLED=false` and use a dedicated database/user.

## Completion Criteria
- `langfuse.adaptdev.ai` serves Langfuse over HTTPS.
- Web and worker services are enabled and healthy under systemd.
- Postgres, ClickHouse, Redis, and R2 connectivity are confirmed from the running system.
- Public signup is disabled.
- Initial owner account and project are provisioned.
- A live ingestion request succeeds and appears in ClickHouse.
- `ops/operations_history.md`, `ops/decisions.log`, and task `completion_report.md` are updated.
- Changes are committed and pushed from branch `working`.

