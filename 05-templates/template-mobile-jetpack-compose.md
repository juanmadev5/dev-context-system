---
tags: [templates, jetpack-compose]
---

# Template — Mobile app (Android / Jetpack Compose)

For a full walk-through of how this file is meant to be used, see [[05-templates/how-to-compose-claude-md|how-to-compose-claude-md]]. Copy the block below into the project's `CLAUDE.md`. The ASP.NET Core imports below are the example variant — swap in [[03-backend/spring-boot/spring-boot|spring-boot]]'s `INDEX.md` instead if that's the project's backend.

---

# <Project Name>

<one-line description of what this project is>

## Context imports

@/home/juanma/dev-context-system/00-global/git-conventions.md
@/home/juanma/dev-context-system/01-mobile/jetpack-compose/INDEX.md

@/home/juanma/dev-context-system/03-backend/aspnet-core/INDEX.md
@/home/juanma/dev-context-system/04-infra/postgresql.md
@/home/juanma/dev-context-system/04-infra/keycloak-auth.md
@/home/juanma/dev-context-system/04-infra/redis.md
@/home/juanma/dev-context-system/04-infra/aws-s3-storage.md
@/home/juanma/dev-context-system/04-infra/docker.md
@/home/juanma/dev-context-system/04-infra/local-infrastructure.md

## Project-specific context

- Domain / purpose: ...
- Minimum supported Android version: ...
- Deviations from the vault's Jetpack Compose defaults for this project, and why (if any): ...

---

## See also

- [[01-mobile/jetpack-compose/jetpack-compose|jetpack-compose]]
- [[05-templates/how-to-compose-claude-md|how-to-compose-claude-md]]
