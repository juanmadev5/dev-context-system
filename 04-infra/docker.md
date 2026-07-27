---
tags: [infra, docker]
---

# Docker

Used for local development on every project, and for the backend **in production too** (not just dev) — see [[tech-stack-map]]. Web frontends deploy to Vercel instead — see [[deployments]].

## Local development

- Shared local services (PostgreSQL, Redis, Keycloak) run from a single always-on stack — see [[local-infrastructure]] — rather than each project spinning up its own copies of these.
- Don't duplicate Postgres/Redis/Keycloak containers per project unless there's a real reason (conflicting versions, isolation requirement) to diverge from the shared stack.

## Root `docker-compose.yml` for multi-service projects

- **Mandatory** whenever a project has more than one runnable service of its own — a backend + frontend, multiple backends, multiple frontends, etc. Purpose: `docker compose up` from the project root brings up every one of the project's *own* services in one shot, purely to make local execution easier.
- **Only the project's own services go in it** — backend(s), frontend(s), nothing else. **Never** PostgreSQL, Redis, or Keycloak in this file: those are assumed to already be running via the shared [[local-infrastructure]] stack, and the project's own services connect to them the same way they would if run directly on the host (`localhost` + published ports, or `host.docker.internal` from inside a container — see [[local-infrastructure]]'s "How a project connects").
- A project with only one runnable service (e.g. Supabase-backed frontend with no dedicated backend) doesn't need a root compose file for this purpose — there's nothing to orchestrate together.

## Production (backend)

- The backend (see [[aspnet-core]], [[spring-boot]]) always ships as a Docker image, in dev and prod alike — no "runs fine on my machine" divergence between environments.
- Multi-stage Dockerfiles: a build stage (SDK image for ASP.NET Core; a Maven/JDK image for Spring Boot) producing the compiled output, copied into a slim runtime-only final stage (ASP.NET runtime image, or a JRE-only image for Spring Boot — not the SDK/JDK image) to keep the production image small.
- Configuration via environment variables / mounted secrets, never baked into the image.
- Containers run as a non-root user (matching the pattern already used in [[local-infrastructure]]'s `docker-compose.yml`).

## Conventions

- `.env` files hold secrets/config and are never committed with real credentials — only committed as `.env.example` templates.
- Image tags are explicit and pinned (avoid bare `latest` in anything beyond throwaway local experiments).

## See also

- [[local-infrastructure]], [[deployments]]
- [[aspnet-core]], [[spring-boot]], [[readme-conventions]]
