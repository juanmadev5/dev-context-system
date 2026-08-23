---
tags: [global, documentation]
---

# README Conventions

Every project has a root `README.md`. Regardless of stack — this is not stack-specific, it's the minimum any human (or agent) needs to get the project running without archaeology.

## Root README — minimum structure

1. **Title + one-line description** — what the project is.
2. **Stack** — brief bullets (frontend/backend/DB/infra). Just enough to orient without opening `CLAUDE.md` or the vault — don't duplicate the detail that already lives there.
3. **Prerequisites** — concrete versions needed to run the project (Node/pnpm, .NET SDK, Docker, Postgres, etc.).
4. **Getting started** — steps to bring the whole thing up end to end. For a multi-service project (backend + frontend, or more), this is `docker compose up` from the root — see [[04-infra/docker|docker]]'s root `docker-compose.yml` convention. If a given service needs its own specific setup beyond that (e.g. a backend service's own config-with-secrets template — see that project's stack note for the exact file), link to that service's own README instead of inlining the detail here.
5. **Environment variables / configuration** — what exists and where it's documented (point to `.env.example` / the `appsettings` template), not the actual values.
6. **Available commands** — dev/build/test/lint, per service if the project has more than one.
7. **Project structure** — only if it isn't obvious at a glance (a 2-level tree is usually enough for a backend+frontend monorepo).

Deployment steps and license are deliberately **not** part of the minimum — add them only if the project genuinely needs them (an internal/private project usually doesn't). Same principle as any stack's Scope discipline rule: don't document what isn't there yet.

## Per-service README (`backend/README.md`, `frontend/README.md`, etc.)

- Created **only when that service has something specific to document that would clutter the root README** — never by default, same reasoning as the "no empty folders" rule every stack's coding-standards note carries, applied to documentation: don't create a file with nothing to say.
- The canonical example: a backend service's own config-with-secrets template (e.g. `appsettings.json`/`appsettings.Development.json`, `application.yml`/`application-dev.yml` — see that project's stack note for the exact convention) — that's genuinely backend-specific and would be noise at the root.
- If a service has nothing specific beyond what the root README already covers, it doesn't get its own README.

## See also

- [[04-infra/docker|docker]]
- [[00-global/git-conventions|git-conventions]]
