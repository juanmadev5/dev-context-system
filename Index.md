---
tags: [index, home]
---

# Dev Context System — Index

This vault is the single source of truth for how AI coding agents (Claude Code and others) should work across every software project. Notes here are written to be imported directly into a project's `CLAUDE.md` via Claude Code's `@path` import syntax — not just read by a human.

## Start here

- **New project?** → [[00-global/tech-stack-map|tech-stack-map]] to pick the stack, then [[05-templates/how-to-compose-claude-md|how-to-compose-claude-md]] to generate that project's `CLAUDE.md`.
- **Changing a rule?** Edit the note here, once. Every project that imports it picks up the change automatically at its next Claude Code session — see [[05-templates/how-to-compose-claude-md|how-to-compose-claude-md]] for why no export/sync step is needed.

## Structure

- `00-global/` — the notes that stay genuinely stack-agnostic: [[00-global/git-conventions|git-conventions]], [[00-global/readme-conventions|readme-conventions]], [[00-global/tech-stack-map|tech-stack-map]] (decision map, used once at setup time).
- `01-mobile/` — [[01-mobile/flutter/INDEX|flutter]], [[01-mobile/jetpack-compose/INDEX|jetpack-compose]]. Each stack folder bundles its own architecture, coding-standards, code-review, responsive-design, and sources notes behind a single `INDEX.md` a project imports.
- `02-web/` — [[02-web/angular/INDEX|angular]], [[02-web/vuejs/INDEX|vuejs]], [[02-web/react/INDEX|react]], plus the shared [[02-web/tailwind-css|tailwind-css]] (not a stack of its own — imported alongside whichever web stack is chosen).
- `03-backend/` — [[03-backend/aspnet-core/INDEX|aspnet-core]], [[03-backend/spring-boot/INDEX|spring-boot]].
- `04-infra/` — [[04-infra/docker|docker]], [[04-infra/deployments|deployments]], [[04-infra/postgresql|postgresql]], [[04-infra/redis|redis]], [[04-infra/keycloak-auth|keycloak-auth]], [[04-infra/aws-s3-storage|aws-s3-storage]], [[04-infra/local-infrastructure|local-infrastructure]].
- `05-templates/` — [[05-templates/how-to-compose-claude-md|how-to-compose-claude-md]] and ready-made `CLAUDE.md` templates per common stack combo.

## Status

All stack notes have their key technical decisions resolved (state management, DI, navigation, validation, testing, etc. per stack). If a project needs to deviate from one of these defaults, document the deviation and the reason in that project's `CLAUDE.md` — and if the deviation turns out to be a standing preference rather than a one-off, update the note in that stack's own folder instead so it's captured once and reused everywhere that stack is used.
