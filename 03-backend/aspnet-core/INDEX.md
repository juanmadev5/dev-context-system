---
tags: [aspnet-core, index]
---

# ASP.NET Core — Index

Single entry point for an ASP.NET Core project's `CLAUDE.md`. Importing this one file (via `@path`) pulls in every rule this stack needs — architecture, coding standards, code review, and how to record consulted documentation — with nothing from any other stack in this repository.

An ASP.NET Core project's `CLAUDE.md` should also import [git-conventions](../../00-global/git-conventions.md) (genuinely stack-agnostic, stays global) and whichever infra notes it actually pairs with. This stack most commonly pairs with [postgresql](../../04-infra/postgresql.md), [keycloak-auth](../../04-infra/keycloak-auth.md), [redis](../../04-infra/redis.md), [aws-s3-storage](../../04-infra/aws-s3-storage.md), and [docker](../../04-infra/docker.md) — see [aspnet-core.md](aspnet-core.md)'s own "Pairs with" section.

Content imports (in order):

@architecture-principles.md
@coding-standards.md
@code-review.md
@sources.md
@aspnet-core.md
