---
tags: [spring-boot, index]
---

# Spring Boot — Index

Single entry point for a Spring Boot project's `CLAUDE.md`. Importing this one file (via `@path`) pulls in every rule this stack needs — architecture, coding standards, code review, and how to record consulted documentation — with nothing from any other stack in this vault.

A Spring Boot project's `CLAUDE.md` should import this file plus [[00-global/git-conventions|git-conventions]] (genuinely stack-agnostic, stays global) and whichever infra notes the project actually pairs with — commonly [[04-infra/postgresql|postgresql]], [[04-infra/keycloak-auth|keycloak-auth]], [[04-infra/redis|redis]], [[04-infra/aws-s3-storage|aws-s3-storage]], and [[04-infra/docker|docker]].

Content imports (in order):

@/home/juanma/dev-context-system/03-backend/spring-boot/architecture-principles.md
@/home/juanma/dev-context-system/03-backend/spring-boot/coding-standards.md
@/home/juanma/dev-context-system/03-backend/spring-boot/code-review.md
@/home/juanma/dev-context-system/03-backend/spring-boot/sources.md
@/home/juanma/dev-context-system/03-backend/spring-boot/spring-boot.md
