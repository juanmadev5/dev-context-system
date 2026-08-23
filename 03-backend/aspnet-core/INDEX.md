---
tags: [aspnet-core, index]
---

# ASP.NET Core — Index

Single entry point for an ASP.NET Core project's `CLAUDE.md`. Importing this one file (via `@path`) pulls in every rule this stack needs — architecture, coding standards, code review, and how to record consulted documentation — with nothing from any other stack in this vault.

An ASP.NET Core project's `CLAUDE.md` should also import [[00-global/git-conventions|git-conventions]] (genuinely stack-agnostic, stays global) and whichever infra notes it actually pairs with. This stack most commonly pairs with [[04-infra/postgresql|postgresql]], [[04-infra/keycloak-auth|keycloak-auth]], [[04-infra/redis|redis]], [[04-infra/aws-s3-storage|aws-s3-storage]], and [[04-infra/docker|docker]] — see [[03-backend/aspnet-core/aspnet-core|aspnet-core.md]]'s own "Pairs with" section.

Content imports (in order):

@C:/Users/juan.velazquez/dev-context-system/03-backend/aspnet-core/architecture-principles.md
@C:/Users/juan.velazquez/dev-context-system/03-backend/aspnet-core/coding-standards.md
@C:/Users/juan.velazquez/dev-context-system/03-backend/aspnet-core/code-review.md
@C:/Users/juan.velazquez/dev-context-system/03-backend/aspnet-core/sources.md
@C:/Users/juan.velazquez/dev-context-system/03-backend/aspnet-core/aspnet-core.md
