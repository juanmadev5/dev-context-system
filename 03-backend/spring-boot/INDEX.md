---
tags: [spring-boot, index]
---

# Spring Boot — Index

Single entry point for a Spring Boot project's `CLAUDE.md`. Importing this one file (via `@path`) pulls in every rule this stack needs — architecture, coding standards, code review, and how to record consulted documentation — with nothing from any other stack in this repository.

A Spring Boot project's `CLAUDE.md` should import this file plus [git-conventions](../../00-global/git-conventions.md) (genuinely stack-agnostic, stays global) and whichever infra notes the project actually pairs with — commonly [postgresql](../../04-infra/postgresql.md), [keycloak-auth](../../04-infra/keycloak-auth.md), [redis](../../04-infra/redis.md), [aws-s3-storage](../../04-infra/aws-s3-storage.md), and [docker](../../04-infra/docker.md).

Content imports (in order):

@architecture-principles.md
@coding-standards.md
@code-review.md
@sources.md
@spring-boot.md
