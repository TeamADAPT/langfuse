# Operations History

## 2026-06-28 07:07:18 — Codex
Prepared Langfuse runtime secrets and database resources. Backed up `/adapt/secrets/.env` and `/adapt/secrets/db.env`, wrote Langfuse env blocks, created the `langfuse` Postgres role/database, created the `langfuse` ClickHouse database/user, and verified Postgres, ClickHouse native/HTTP, and Redis connectivity from user `x`.

Files touched:
- `/adapt/secrets/.env`
- `/adapt/secrets/db.env`
- `/adapt/secrets/.env.bak-langfuse-*`
- `/adapt/secrets/db.env.bak-langfuse-*`
- `ops/operations_history.md`
- `ops/decisions.log`
- `ops/in_progress/langfuse-production-systemd-install/task.md`

## 2026-06-28 07:03:58 — Codex
Received approval for the Langfuse production systemd install plan and updated the reverse proxy plan from Caddy to nginx, with Cloudflare Tunnel fallback credentials located at `/adapt/secrets/m2.env`.

Files touched:
- `plans/langfuse-production-systemd-install.md`
- `ops/operations_history.md`
- `ops/decisions.log`
- `ops/in_progress/langfuse-production-systemd-install/task.md`

## 2026-06-28 06:39:59 — Codex
Attempted the planning checkpoint commit; Git rejected it because no author identity is configured for this repository or user. Retrying with a per-command author identity only.

Files touched:
- `ops/operations_history.md`
- `ops/decisions.log`

## 2026-06-28 06:38:57 — Codex
Initialized Langfuse production systemd install task scaffolding and wrote the approval plan before runtime changes.

Files touched:
- `ops/operations_history.md`
- `ops/decisions.log`
- `ops/in_progress/langfuse-production-systemd-install/task.md`
- `plans/langfuse-production-systemd-install.md`
