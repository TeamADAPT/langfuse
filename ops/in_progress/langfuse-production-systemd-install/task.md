# Langfuse Production Systemd Install Task

## 2026-06-28 06:38:57 — Codex
Task started for production single-node Langfuse install at `langfuse.adaptdev.ai`.

Scope:
- Install and run Langfuse from this repository with systemd.
- Use existing local Postgres, ClickHouse, and Redis where viable.
- Configure Cloudflare R2 as S3-compatible blob/event storage.
- Disable public signups.
- Verify with live running-system receipts.

Current status:
- Approved for execution.
- Reverse proxy target is nginx.
- Runtime secrets and dedicated Postgres/ClickHouse resources are prepared.
- Node.js 24, nginx, dependencies, and ClickHouse migrate CLI are installed.
- Web and worker production builds completed.
- Postgres and ClickHouse migrations completed via `langfuse-migrate.service`.
- `langfuse-web.service`, `langfuse-worker.service`, `langfuse-redis.service`, `langfuse-cloudflared.service`, `nginx.service`, and `langfuse.target` are active.
- Public HTTPS health receipt: `https://langfuse.adaptdev.ai/api/public/health?failIfDatabaseUnavailable=true` returned `200 OK` and `{"status":"OK","version":"3.201.1"}`.
- Cloudflare Tunnel `langfuse-adaptdev-ai` is healthy with 4 connections and proxied DNS CNAME `langfuse.adaptdev.ai`.
- Signup-disabled receipt: `POST /api/auth/signup` returned `422 {"message":"Sign up is disabled."}`.
- Database initialization receipt: Postgres has 1 user, 1 organization, 1 project, and 1 API key; ClickHouse HTTP query returned `1`.
- R2 verification is blocked by Cloudflare API error `10042` until R2 is enabled for the account.
- Local Turbopack build shim `web/packages -> ../packages` is present and ignored by git.
