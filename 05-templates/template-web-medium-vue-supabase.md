---
tags: [templates, vuejs, supabase]
---

# Template — Medium/small web app (Vue + Supabase)

For a full walk-through of how this file is meant to be used, see [[how-to-compose-claude-md]]. Copy the block below into the project's `CLAUDE.md`.

---

# <Project Name>

<one-line description of what this project is>

## Context imports

@/home/juanma/Documents/dev-context-system/00-global/coding-standards.md
@/home/juanma/Documents/dev-context-system/00-global/architecture-principles.md
@/home/juanma/Documents/dev-context-system/00-global/git-conventions.md
@/home/juanma/Documents/dev-context-system/00-global/responsive-design.md
@/home/juanma/Documents/dev-context-system/00-global/sources-conventions.md
@/home/juanma/Documents/dev-context-system/02-web/vuejs.md
@/home/juanma/Documents/dev-context-system/02-web/tailwind-css.md
@/home/juanma/Documents/dev-context-system/03-backend/supabase.md
@/home/juanma/Documents/dev-context-system/04-infra/postgresql.md
@/home/juanma/Documents/dev-context-system/04-infra/supabase-auth.md
@/home/juanma/Documents/dev-context-system/04-infra/supabase-storage.md
@/home/juanma/Documents/dev-context-system/04-infra/deployments.md

## Project-specific context

- Domain / purpose: ...
- Deviations from the vault's Vue/Supabase defaults for this project, and why (if any): ...

---

## Variant — Vue + ASP.NET Core backend

If this medium-sized project needs a dedicated backend instead of Supabase, swap the backend imports:

```
@/home/juanma/Documents/dev-context-system/03-backend/aspnet-core.md
@/home/juanma/Documents/dev-context-system/04-infra/keycloak-auth.md
@/home/juanma/Documents/dev-context-system/04-infra/redis.md
@/home/juanma/Documents/dev-context-system/04-infra/aws-s3-storage.md
@/home/juanma/Documents/dev-context-system/04-infra/docker.md
@/home/juanma/Documents/dev-context-system/04-infra/local-infrastructure.md
```

in place of the `supabase*` imports above.

## See also

- [[vuejs]], [[supabase]]
- [[how-to-compose-claude-md]]
