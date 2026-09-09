---
tags: [aspnet-core, index]
---

# ASP.NET Core — Index

Copy manifest for an ASP.NET Core project. Once ASP.NET Core is the chosen stack (see [tech-stack-map](../../00-global/tech-stack-map.md)), copy every file listed below into that project's own `/docs/` folder — this is everything the stack needs: architecture, coding standards, code review, and how to record consulted documentation. Nothing from any other stack in this repository is needed.

Also copy [git-conventions](../../00-global/git-conventions.md) (genuinely stack-agnostic, stays global), [rest-api-design](../rest-api-design.md) (if the project exposes a REST API), and whichever infra notes it actually pairs with. This stack most commonly pairs with [postgresql](../../04-infra/postgresql.md), [keycloak-auth](../../04-infra/keycloak-auth.md), [redis](../../04-infra/redis.md), [aws-s3-storage](../../04-infra/aws-s3-storage.md), and [docker](../../04-infra/docker.md) — see [aspnet-core.md](aspnet-core.md)'s own "Pairs with" section. The project's own `CLAUDE.md` then points at `/docs/` instead of at this repository.

Files to copy (read in this order):

1. architecture-principles.md
2. coding-standards.md
3. code-review.md
4. sources.md
5. aspnet-core.md
