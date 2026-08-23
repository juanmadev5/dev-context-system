---
tags: [infra, docker]
---

# Docker

Used for local development on every project, and for the backend **in production too** (not just dev) — see [[00-global/tech-stack-map|tech-stack-map]]. Web frontends deploy to Vercel instead — see [[04-infra/deployments|deployments]].

## Local development

- Shared local services (PostgreSQL, Redis, Keycloak) run from a single always-on stack — see [[04-infra/local-infrastructure|local-infrastructure]] — rather than each project spinning up its own copies of these.
- Don't duplicate Postgres/Redis/Keycloak containers per project unless there's a real reason (conflicting versions, isolation requirement) to diverge from the shared stack.

## Root `docker-compose.yml` for multi-service projects

- **Mandatory** whenever a project has more than one runnable service of its own — a backend + frontend, multiple backends, multiple frontends, etc. Purpose: `docker compose up` from the project root brings up every one of the project's *own* services in one shot, purely to make local execution easier.
- **Only the project's own services go in it** — backend(s), frontend(s), nothing else. **Never** PostgreSQL, Redis, or Keycloak in this file: those are assumed to already be running via the shared [[04-infra/local-infrastructure|local-infrastructure]] stack, and the project's own services connect to them the same way they would if run directly on the host (`localhost` + published ports, or `host.docker.internal` from inside a container — see [[04-infra/local-infrastructure|local-infrastructure]]'s "How a project connects").
- The compose file doesn't replace each service's own Dockerfile — it references it via `build.context` (e.g. `context: ./backend` points at `backend/Dockerfile`, the multi-stage build already described below). Compose's job is orchestration: build the images, start the containers together, and wire the network between them (each service is reachable from another by its service name as hostname, e.g. `http://backend:8080` from inside the frontend container).
- **Each service gets its own `.env`, never a single root `.env` shared across all of them** — `backend/.env`, `frontend/.env`, etc., each wired into that service only via `env_file: .env` in its own service definition. `env_file` dumps every key in that file into the container with no filtering, so a shared root `.env` would leak the backend's secrets into the frontend's container (and, for a browser-executed frontend, into whatever a bundler happens to inline) even though the frontend code never references them. Compose's own automatic root-level `.env` lookup is a *different* mechanism — it only substitutes `${VAR}` placeholders written inside `docker-compose.yml` itself (image tags, port numbers) and does **not** inject anything into a container; don't rely on it as a substitute for `env_file`.
- A project with only one runnable service doesn't need a root compose file for this purpose — there's nothing to orchestrate together. It still gets its own `.env`, run via `docker run --env-file .env` (or the single service's `env_file:` entry if a minimal compose file is used anyway for the `docker compose up` convenience).

## Production (backend)

- The backend (see [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]) always ships as a Docker image, in dev and prod alike — no "runs fine on my machine" divergence between environments.
- Multi-stage Dockerfiles: a build stage (SDK image for ASP.NET Core; a Maven/JDK image for Spring Boot) producing the compiled output, copied into a slim runtime-only final stage (ASP.NET runtime image, or a JRE-only image for Spring Boot — not the SDK/JDK image) to keep the production image small.
- Configuration via environment variables / mounted secrets, never baked into the image.
- Containers run as a non-root user (matching the pattern already used in [[04-infra/local-infrastructure|local-infrastructure]]'s `docker-compose.yml`).

## Conventions

- Each service's `.env` is never committed with real credentials — only committed as that service's own `.env.example` template (`backend/.env.example`, `frontend/.env.example`).
- Image tags are explicit and pinned (avoid bare `latest` in anything beyond throwaway local experiments).

## See also

- [[04-infra/local-infrastructure|local-infrastructure]], [[04-infra/deployments|deployments]]
- [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]], [[00-global/readme-conventions|readme-conventions]]
