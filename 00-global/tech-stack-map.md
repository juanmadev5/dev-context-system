---
tags: [global, index, tech-stack]
---

# Tech Stack Map

Decision map: given a project's shape, which technology to reach for. This is the entry point an agent should read first to understand *why* a given stack was chosen for a project, and it's also what a human uses to pick the stack for a new project.

## Frontend / client

| Project shape | Choice | Notes |
| --- | --- | --- |
| Mobile app | [flutter](../01-mobile/flutter/flutter.md) (cross-platform) or [jetpack-compose](../01-mobile/jetpack-compose/jetpack-compose.md) (Android-native) | Compose when the project is Android-only and needs deep platform integration; Flutter otherwise. |
| Complex web app, heavy business logic | [angular](../02-web/angular/angular.md) | Structured, opinionated, scales with team size and complexity. Pairs with Clean Architecture — see that note's own `architecture-principles.md`. |
| Medium/small web app, or a simple landing page | [vuejs](../02-web/vuejs/vuejs.md) or [react](../02-web/react/react.md) | Equivalent defaults, pick per the team/project's existing preference — neither is a "smaller" choice. Less ceremony than Angular. Pairs with Vertical Slice or plain structure. |
| Styling, any of the above | [tailwind-css](../02-web/tailwind-css.md) | Always Tailwind, no exceptions. |

### Package manager (Node.js projects)

- **pnpm**, always — never `npm` or `yarn`. Applies to every project with a `package.json`: [angular](../02-web/angular/angular.md), [vuejs](../02-web/vuejs/vuejs.md), [react](../02-web/react/react.md) (Tailwind CLI/build tooling), and any Node-based tooling in general.
- Lockfile is `pnpm-lock.yaml`, committed to the repo. Don't let a stray `package-lock.json`/`yarn.lock` creep back in.
- Scripts/docs/CI reference `pnpm install`, `pnpm run <script>` (or the `pnpm <script>` shorthand) — never `npm install`/`npm run`.

## Backend

| Project shape | Choice | Notes |
| --- | --- | --- |
| Every project that needs a backend | [aspnet-core](../03-backend/aspnet-core/aspnet-core.md) or [spring-boot](../03-backend/spring-boot/spring-boot.md) | Equivalent defaults, pick per the team/project's existing language ecosystem (.NET vs Java) — neither is a "smaller" choice. Default even for small APIs. ASP.NET Core still has a size call (Minimal APIs vs. Controllers, Clean Architecture vs. Vertical Slice — see that stack's own note); Spring Boot always uses Clean Architecture. |

## Data & infra

| Concern | Choice |
| --- | --- |
| Database | [postgresql](../04-infra/postgresql.md) — always, for every project. |
| Cache | [redis](../04-infra/redis.md) |
| Auth | [keycloak-auth](../04-infra/keycloak-auth.md) |
| File storage | [aws-s3-storage](../04-infra/aws-s3-storage.md) |
| Local dev services (Postgres, Redis, Keycloak) | [local-infrastructure](../04-infra/local-infrastructure.md) |
| Containerization | [docker](../04-infra/docker.md) — local dev always; backend in production too. |
| Deployment | [docker](../04-infra/docker.md) — see its "Production" section |

## Composing a project's context

Once the stack for a given project is picked from the tables above, list that stack's own folder (e.g. `01-mobile/flutter/`) and copy every file in it into that project's own `/docs/` folder, plus [git-conventions](git-conventions.md) and whichever infra notes the project pairs with. Each stack's architecture, coding-standards, code-review, responsive-design (where applicable), and sources conventions live inside that stack's own folder, not as separate global notes. The project's own `CLAUDE.md` then points at `/docs/` instead of at this repository — see the root [README](../README.md)'s "How an agent should use this repository".
