---
tags: [global, index, tech-stack]
---

# Tech Stack Map

Decision map: given a project's shape, which technology to reach for. This is the entry point an agent should read first to understand *why* a given stack was chosen for a project, and it's also what a human uses to pick the stack for a new project.

## Frontend / client

| Project shape | Choice | Notes |
| --- | --- | --- |
| Mobile app | [[flutter]] (cross-platform) or [[jetpack-compose]] (Android-native) | Compose when the project is Android-only and needs deep platform integration; Flutter otherwise. |
| Complex web app, heavy business logic | [[angular]] | Structured, opinionated, scales with team size and complexity. Pairs with Clean Architecture — see [[architecture-principles]]. |
| Medium/small web app | [[vuejs]] or [[react]] | Equivalent defaults, pick per the team/project's existing preference — neither is a "smaller" choice, same reasoning as the backend's ASP.NET Core/Spring Boot pair. Less ceremony than Angular. Pairs with Vertical Slice or plain structure. |
| Landing page, medium/large, some interactivity | [[astro]] | Content-first, islands architecture for the interactive bits. |
| Simple landing page, little to no functionality | [[html-tailwind]] | Plain HTML when a framework would be overhead. |
| Styling, any of the above | [[tailwind-css]] | Always Tailwind, no exceptions. |

### Package manager (Node.js projects)

- **pnpm**, always — never `npm` or `yarn`. Applies to every project with a `package.json`: [[angular]], [[vuejs]], [[react]], [[astro]], [[html-tailwind]] (Tailwind CLI/build tooling), and any Node-based tooling in general.
- Lockfile is `pnpm-lock.yaml`, committed to the repo. Don't let a stray `package-lock.json`/`yarn.lock` creep back in.
- Scripts/docs/CI reference `pnpm install`, `pnpm run <script>` (or the `pnpm <script>` shorthand) — never `npm install`/`npm run`.

## Backend

| Project shape | Choice | Notes |
| --- | --- | --- |
| Needs a dedicated backend (custom business logic, non-trivial auth, integration with S3/Keycloak/etc.) | [[aspnet-core]] or [[spring-boot]] | Equivalent defaults, pick per the team/project's existing language ecosystem (.NET vs Java) — neither is a "smaller" choice. Default even for small APIs, not a reason to reach for something else. Minimal APIs vs. Controllers (ASP.NET Core) or the Clean Architecture vs. package-by-feature call (Spring Boot) is a size call — see that stack's own note. |
| App is simple enough that a dedicated backend is overkill | [[supabase]] | Postgres + auth + storage + realtime out of the box. |

## Data & infra

| Concern | Choice |
| --- | --- |
| Database | [[postgresql]] — always, for every project. |
| Cache | [[redis]] |
| Auth (with a dedicated ASP.NET Core/Spring Boot backend) | [[keycloak-auth]] |
| Auth (Supabase-only project) | [[supabase-auth]] |
| File storage (with a dedicated ASP.NET Core/Spring Boot backend) | [[aws-s3-storage]] |
| File storage (Supabase-only project) | [[supabase-storage]] |
| Local dev services (Postgres, Redis, Keycloak) | [[local-infrastructure]] |
| Containerization | [[docker]] — local dev always; backend in production too. |
| Deployment | [[deployments]] |

## Composing a project's context

Once the stack for a given project is picked from the tables above, see [[how-to-compose-claude-md]] to generate that project's `CLAUDE.md` by importing the relevant notes from this vault.

## See also

- [[coding-standards]]
- [[architecture-principles]]
- [[responsive-design]] — applies to every frontend entry in the table above, web or mobile.
