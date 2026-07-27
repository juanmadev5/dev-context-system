---
tags: [infra, docker, local-dev]
---

# Local Infrastructure

Shared Docker Compose stack providing PostgreSQL, Keycloak, and Redis for local development across projects, so individual projects connect via `localhost` instead of each spinning up their own copies.

Location: `~/dev/local-infrastructure` (see that repo's `README.md` for the full, authoritative setup/troubleshooting guide — this note only captures what an agent needs to know to reason about a project connecting to it).

## Services

| Service | Image | Default host port | Persistence |
|---|---|---|---|
| PostgreSQL | Postgres 16 | `5432` (`POSTGRES_PORT`) | volume `postgres_data` |
| Keycloak | Keycloak 26.x, `start-dev` mode | `8080` (`KEYCLOAK_PORT`) | volume `keycloak_data` + `keycloak` DB in Postgres |
| Redis | Redis 8.x | `6379` (`REDIS_PORT`) | volume `redis_data`, AOF persistence |

- All configured via `.env` in that repo (`POSTGRES_USER/PASSWORD/DB`, ports, `KC_BOOTSTRAP_ADMIN_USERNAME/PASSWORD`).
- Containers run as non-root users.
- The `keycloak` Postgres database is reserved for Keycloak's own state — application data always lives in a separate database (`POSTGRES_DB`, or one created per project), never mixed with `keycloak`.

## How a project connects

- **From the host** (an app run directly, not in Docker): `localhost` + the published port for each service. See [[postgresql]], [[redis]], [[keycloak-auth]] for stack-specific conventions on top of this.
- **From inside another project's own Docker Compose stack**: `localhost` inside that container refers to the container itself, not this stack. Either connect via `host.docker.internal` (with `extra_hosts` on Linux) + published ports, or join a shared external Docker network (`docker network create local-shared`) if that project's compose file is deliberately configured for it. Default to host + published ports unless a project has a specific reason to need the shared-network topology.

## Lifecycle

- `docker compose up -d` / `down` / `down -v` (the latter wipes all data — Postgres, Keycloak, Redis alike) from within `~/dev/local-infrastructure`.
- `start-dev` Keycloak mode is dev-only; never used as a production template — see [[keycloak-auth]].

## See also

- [[docker]], [[postgresql]], [[redis]], [[keycloak-auth]]
