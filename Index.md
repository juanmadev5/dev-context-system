---
tags: [index, home]
---

# Dev Context System — Index

This repository is the single source of truth for how AI coding agents (Claude Code and others) should work across every software project. Notes here are written to be imported directly into a project's `CLAUDE.md` via Claude Code's `@path` import syntax — not just read by a human.

## Start here

- **New project?** → [tech-stack-map](00-global/tech-stack-map.md) to pick the stack, then compose that project's `CLAUDE.md` by importing the chosen stack's `INDEX.md` (see `CLAUDE.md`'s "The `@path` import mechanism" section in this repo).
- **Changing a rule?** Edit the note here, once. Every project that imports it picks up the change automatically at its next Claude Code session — no export/sync step is needed.

## Structure

- `00-global/` — the notes that stay genuinely stack-agnostic: [git-conventions](00-global/git-conventions.md), [readme-conventions](00-global/readme-conventions.md), [tech-stack-map](00-global/tech-stack-map.md) (decision map, used once at setup time).
- `01-mobile/` — [flutter](01-mobile/flutter/INDEX.md), [jetpack-compose](01-mobile/jetpack-compose/INDEX.md). Each stack folder bundles its own architecture, coding-standards, code-review, responsive-design, and sources notes behind a single `INDEX.md` a project imports.
- `02-web/` — [angular](02-web/angular/INDEX.md), [vuejs](02-web/vuejs/INDEX.md), [react](02-web/react/INDEX.md), plus the shared [tailwind-css](02-web/tailwind-css.md) (not a stack of its own — imported alongside whichever web stack is chosen).
- `03-backend/` — [aspnet-core](03-backend/aspnet-core/INDEX.md), [spring-boot](03-backend/spring-boot/INDEX.md).
- `04-infra/` — [docker](04-infra/docker.md), [postgresql](04-infra/postgresql.md), [redis](04-infra/redis.md), [keycloak-auth](04-infra/keycloak-auth.md), [aws-s3-storage](04-infra/aws-s3-storage.md), [local-infrastructure](04-infra/local-infrastructure.md).

## Status

All stack notes have their key technical decisions resolved (state management, DI, navigation, validation, testing, etc. per stack). If a project needs to deviate from one of these defaults, document the deviation and the reason in that project's `CLAUDE.md` — and if the deviation turns out to be a standing preference rather than a one-off, update the note in that stack's own folder instead so it's captured once and reused everywhere that stack is used.
