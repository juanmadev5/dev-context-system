---
tags: [global, index, tech-stack]
---

# Tech Stack Map

Decision map: given a project's shape, which technology to reach for. This is the entry point an agent should read first to understand *why* a given stack was chosen for a project, and it's also what a human uses to pick the stack for a new project.

## Frontend / client

| Project shape | Choice | Notes |
| --- | --- | --- |
| Mobile app | [[01-mobile/flutter/flutter|flutter]] (cross-platform) or [[01-mobile/jetpack-compose/jetpack-compose|jetpack-compose]] (Android-native) | Compose when the project is Android-only and needs deep platform integration; Flutter otherwise. |
| Complex web app, heavy business logic | [[02-web/angular/angular|angular]] | Structured, opinionated, scales with team size and complexity. Pairs with Clean Architecture — see that note's own `architecture-principles.md`. |
| Medium/small web app, or a simple landing page | [[02-web/vuejs/vuejs|vuejs]] or [[02-web/react/react|react]] | Equivalent defaults, pick per the team/project's existing preference — neither is a "smaller" choice. Less ceremony than Angular. Pairs with Vertical Slice or plain structure. |
| Styling, any of the above | [[02-web/tailwind-css|tailwind-css]] | Always Tailwind, no exceptions. |

### Package manager (Node.js projects)

- **pnpm**, always — never `npm` or `yarn`. Applies to every project with a `package.json`: [[02-web/angular/angular|angular]], [[02-web/vuejs/vuejs|vuejs]], [[02-web/react/react|react]] (Tailwind CLI/build tooling), and any Node-based tooling in general.
- Lockfile is `pnpm-lock.yaml`, committed to the repo. Don't let a stray `package-lock.json`/`yarn.lock` creep back in.
- Scripts/docs/CI reference `pnpm install`, `pnpm run <script>` (or the `pnpm <script>` shorthand) — never `npm install`/`npm run`.

## Backend

| Project shape | Choice | Notes |
| --- | --- | --- |
| Every project that needs a backend | [[03-backend/aspnet-core/aspnet-core|aspnet-core]] or [[03-backend/spring-boot/spring-boot|spring-boot]] | Equivalent defaults, pick per the team/project's existing language ecosystem (.NET vs Java) — neither is a "smaller" choice. Default even for small APIs. Minimal APIs vs. Controllers (ASP.NET Core) or the Clean Architecture vs. package-by-feature call (Spring Boot) is a size call — see that stack's own note. |

## Data & infra

| Concern | Choice |
| --- | --- |
| Database | [[04-infra/postgresql|postgresql]] — always, for every project. |
| Cache | [[04-infra/redis|redis]] |
| Auth | [[04-infra/keycloak-auth|keycloak-auth]] |
| File storage | [[04-infra/aws-s3-storage|aws-s3-storage]] |
| Local dev services (Postgres, Redis, Keycloak) | [[04-infra/local-infrastructure|local-infrastructure]] |
| Containerization | [[04-infra/docker|docker]] — local dev always; backend in production too. |
| Deployment | [[04-infra/docker|docker]] — see its "Production" section |

## Composing a project's context

Once the stack for a given project is picked from the tables above, compose that project's `CLAUDE.md` by importing that stack's own `INDEX.md` (e.g. `01-mobile/flutter/INDEX.md`) via Claude Code's `@path` syntax, plus [[00-global/git-conventions|git-conventions]] and whichever infra notes the project pairs with. Each stack's architecture, coding-standards, code-review, responsive-design (where applicable), and sources conventions live inside that stack's own folder, not as separate global notes.

## See also

- [[00-global/git-conventions|git-conventions]]
