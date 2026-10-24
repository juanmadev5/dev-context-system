---
tags: [global, docker]
---

# Docker

Decision rule for when a project needs a `Dockerfile` and when it needs a `docker-compose.yml`. Nothing else — how a project's infrastructure dependencies (database, cache, auth provider, etc.) are run is out of scope here.

## Dockerfile

- Every runnable service the project owns (a backend, a frontend) gets its own `Dockerfile`.

## docker-compose.yml

- **Only when the project has more than one runnable service of its own** — a backend + frontend, multiple backends, multiple frontends, etc. Purpose: `docker compose up` from the project root brings up every one of the project's *own* services in one shot, purely to make local execution easier.
- **Only the project's own services go in it.** Never infrastructure the project depends on but doesn't own (database, cache, auth provider, etc.) — those are assumed to already be running elsewhere, and the project's own services connect to them the same way they would if run directly on the host.
- A project with a single runnable service doesn't need one — there's nothing to orchestrate together.
