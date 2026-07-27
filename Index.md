---
tags: [index, home]
---

# Dev Context System — Index

This vault is the single source of truth for how AI coding agents (Claude Code and others) should work across every software project. Notes here are written to be imported directly into a project's `CLAUDE.md` via Claude Code's `@path` import syntax — not just read by a human.

## Start here

- **New project?** → [[tech-stack-map]] to pick the stack, then [[how-to-compose-claude-md]] to generate that project's `CLAUDE.md`.
- **Changing a rule?** Edit the note here, once. Every project that imports it picks up the change automatically at its next Claude Code session — see [[how-to-compose-claude-md]] for why no export/sync step is needed.

## Structure

- `00-global/` — rules that apply to every project regardless of stack: [[coding-standards]], [[architecture-principles]], [[git-conventions]], [[tech-stack-map]], [[responsive-design]], [[sources-conventions]].
- `01-mobile/` — [[flutter]], [[jetpack-compose]].
- `02-web/` — [[angular]], [[vuejs]], [[react]], [[astro]], [[html-tailwind]], [[tailwind-css]].
- `03-backend/` — [[aspnet-core]], [[spring-boot]], [[supabase]].
- `04-infra/` — [[docker]], [[deployments]], [[postgresql]], [[redis]], [[keycloak-auth]], [[supabase-auth]], [[aws-s3-storage]], [[supabase-storage]], [[local-infrastructure]].
- `05-templates/` — [[how-to-compose-claude-md]] and ready-made `CLAUDE.md` templates per common stack combo.

## Status

All stack notes have their key technical decisions resolved (state management, DI, navigation, validation, testing, etc. per stack). If a project needs to deviate from one of these defaults, document the deviation and the reason in that project's `CLAUDE.md` — and if the deviation turns out to be a standing preference rather than a one-off, update the note here instead so it's captured once and reused everywhere.
