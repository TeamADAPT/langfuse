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
- Cloudflare Tunnel credentials may be used from `/adapt/secrets/m2.env` if no local TLS certificate/key is available.
- Runtime secrets and dedicated Postgres/ClickHouse resources are prepared and connectivity has been verified from user `x`.
- Node.js 24, nginx, dependencies, and ClickHouse migrate CLI are installed.
- R2 verification is blocked by Cloudflare API error `10042` until R2 is enabled for the account.
- Local Turbopack build shim `web/packages -> ../packages` is present and ignored by git.
