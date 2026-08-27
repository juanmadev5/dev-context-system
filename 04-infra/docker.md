---
tags: [infra, docker]
---

# Docker

Used for local development on every project, and **in production too** (not just dev) for both the web frontend and the backend. No serverless/PaaS deployment for either — everything ships as a Docker image.

## Root `docker-compose.yml` for multi-service projects

- **Mandatory** whenever a project has more than one runnable service of its own — a backend + frontend, multiple backends, multiple frontends, etc. Purpose: `docker compose up` from the project root brings up every one of the project's *own* services in one shot, purely to make local execution easier.
- **Only the project's own services go in it** — backend(s), frontend(s), nothing else. **Never** infrastructure services the project depends on but doesn't own (database, cache, auth provider, etc.) — those are assumed to already be running elsewhere, and the project's own services connect to them the same way they would if run directly on the host (`localhost` + published ports, or `host.docker.internal` from inside a container).
- The compose file doesn't replace each service's own Dockerfile — it references it via `build.context` (e.g. `context: ./backend` points at `backend/Dockerfile`, the multi-stage build already described below). Compose's job is orchestration: build the images, start the containers together, and wire the network between them (each service is reachable from another by its service name as hostname, e.g. `http://backend:8080` from inside the frontend container).
- **Each service gets its own `.env`, never a single root `.env` shared across all of them** — `backend/.env`, `frontend/.env`, etc., each wired into that service only via `env_file: .env` in its own service definition. `env_file` dumps every key in that file into the container with no filtering, so a shared root `.env` would leak the backend's secrets into the frontend's container (and, for a browser-executed frontend, into whatever a bundler happens to inline) even though the frontend code never references them. Compose's own automatic root-level `.env` lookup is a *different* mechanism — it only substitutes `${VAR}` placeholders written inside `docker-compose.yml` itself (image tags, port numbers) and does **not** inject anything into a container; don't rely on it as a substitute for `env_file`.
- A project with only one runnable service doesn't need a root compose file for this purpose — there's nothing to orchestrate together. It still gets its own `.env`, run via `docker run --env-file .env` (or the single service's `env_file:` entry if a minimal compose file is used anyway for the `docker compose up` convenience).

## Production

- Both the frontend and the backend always ship as a Docker image, in dev and prod alike — no "runs fine on my machine" divergence between environments. A project with a single runnable service just needs that service's own Dockerfile; a project with more than one runs them together via the root `docker-compose.yml` described above, reused/adapted for the deploy pipeline.
- Multi-stage Dockerfiles: a build stage (the ecosystem's SDK/build-tool image) producing the compiled output, copied into a slim runtime-only final stage (a runtime-only image, never the SDK/build image) to keep the production image small.
- Prefer a deploy pipeline that builds the image(s) from the same Dockerfile(s)/compose file used locally, so dev/prod parity holds.
- Configuration via environment variables / mounted secrets, never baked into the image.
- Containers run as a non-root user.

## Conventions

- Each service's `.env` is never committed with real credentials — only committed as that service's own `.env.example` template (`backend/.env.example`, `frontend/.env.example`).
- Image tags are explicit and pinned (avoid bare `latest` in anything beyond throwaway local experiments).
