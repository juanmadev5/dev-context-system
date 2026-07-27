---
tags: [templates, jetpack-compose]
---

# Template — Mobile app (Android / Jetpack Compose)

For a full walk-through of how this file is meant to be used, see [[how-to-compose-claude-md]]. Copy the block below into the project's `CLAUDE.md`. Pick the backend variant that fits the project.

---

# <Project Name>

<one-line description of what this project is>

## Context imports

@/home/juanma/Documents/dev-context-system/00-global/coding-standards.md
@/home/juanma/Documents/dev-context-system/00-global/architecture-principles.md
@/home/juanma/Documents/dev-context-system/00-global/git-conventions.md
@/home/juanma/Documents/dev-context-system/00-global/responsive-design.md
@/home/juanma/Documents/dev-context-system/00-global/sources-conventions.md
@/home/juanma/Documents/dev-context-system/01-mobile/jetpack-compose.md

### Backend variant — ASP.NET Core

@/home/juanma/Documents/dev-context-system/03-backend/aspnet-core.md
@/home/juanma/Documents/dev-context-system/04-infra/postgresql.md
@/home/juanma/Documents/dev-context-system/04-infra/keycloak-auth.md
@/home/juanma/Documents/dev-context-system/04-infra/redis.md
@/home/juanma/Documents/dev-context-system/04-infra/aws-s3-storage.md
@/home/juanma/Documents/dev-context-system/04-infra/docker.md
@/home/juanma/Documents/dev-context-system/04-infra/local-infrastructure.md

### Backend variant — Supabase

@/home/juanma/Documents/dev-context-system/03-backend/supabase.md
@/home/juanma/Documents/dev-context-system/04-infra/postgresql.md
@/home/juanma/Documents/dev-context-system/04-infra/supabase-auth.md
@/home/juanma/Documents/dev-context-system/04-infra/supabase-storage.md

(Keep only the imports for the variant that applies — remove the other block.)

## Project-specific context

- Domain / purpose: ...
- Minimum supported Android version: ...
- Deviations from the vault's Jetpack Compose defaults for this project, and why (if any): ...

---

## See also

- [[jetpack-compose]]
- [[how-to-compose-claude-md]]
