# Dev Context System

A personal knowledge base of engineering decisions I've already made — architecture, naming, git hygiene, per-stack conventions — so AI coding agents write code the way I actually want it written instead of improvising a fresh, "reasonable" convention on every project.

## What this is

This is **not** a software project — it's a repository of plain Markdown notes, with no application code, no dependency manifest, and no build/lint/test tooling. "Working on it" means editing notes accurately and keeping them internally consistent.

Every note reflects a decision I've actually made and stand behind — never a generic best-practices checklist copied from somewhere else. If a rule is missing, that's a question for me, not something for an agent to infer or fill in.

## How an agent should use this repository

1. **Give the agent read access to this directory.** It doesn't get imported live into a project — it's a library an agent consults once, at project-definition time.
2. **During a new project's Spec-Driven Development phase**, once the project's shape and stack are understood, use [tech-stack-map](00-global/tech-stack-map.md) to confirm (or pick) the stack.
3. **Copy the files that stack needs into the project's own `/docs/` folder.** Each stack folder's `INDEX.md` is a copy manifest — a plain list of exactly which files to copy and in what order to read them (architecture, coding standards, code review, responsive design where applicable, sources, the stack note itself). Also copy:
   - [git-conventions](00-global/git-conventions.md) — every project needs it.
   - [tailwind-css](02-web/tailwind-css.md) — every web project.
   - [rest-api-design](03-backend/rest-api-design.md) — every backend project that exposes a REST API.
   - whichever [04-infra/](04-infra/) notes the project actually pairs with (database, cache, auth, storage, containerization).
4. **The project's own `CLAUDE.md` then points at `/docs/`**, not at this repository — plain relative links to the copied files. From that point on the project is self-contained: no live dependency on this repo, no cross-repo imports. If a rule here changes later, re-copy the affected file into the project when it's relevant to do so.

This deliberately replaces an earlier version of this system that used Claude Code's `@path` import syntax to pull content live into a project's `CLAUDE.md`. That tied every consuming project to one tool's import mechanism; copying plain files into `/docs/` works with any agent that can read a directory.

## Structure

- `00-global/` — the notes that stay genuinely stack-agnostic: [git-conventions](00-global/git-conventions.md), [readme-conventions](00-global/readme-conventions.md), [tech-stack-map](00-global/tech-stack-map.md) (the decision map, read once per project — not copied into a project's `/docs/`).
- `01-mobile/` — [flutter](01-mobile/flutter/INDEX.md), [jetpack-compose](01-mobile/jetpack-compose/INDEX.md).
- `02-web/` — [angular](02-web/angular/INDEX.md), [vuejs](02-web/vuejs/INDEX.md), [react](02-web/react/INDEX.md), plus the shared [tailwind-css](02-web/tailwind-css.md) (not a stack of its own — copied alongside whichever web stack is chosen).
- `03-backend/` — [aspnet-core](03-backend/aspnet-core/INDEX.md), [spring-boot](03-backend/spring-boot/INDEX.md), plus the shared [rest-api-design](03-backend/rest-api-design.md) (not a stack of its own — copied alongside whichever backend stack exposes a REST API).
- `04-infra/` — one self-contained note per infrastructure piece: [docker](04-infra/docker.md), [postgresql](04-infra/postgresql.md), [redis](04-infra/redis.md), [keycloak-auth](04-infra/keycloak-auth.md), [aws-s3-storage](04-infra/aws-s3-storage.md), [local-infrastructure](04-infra/local-infrastructure.md).

Each mobile/web/backend stack folder bundles its own `architecture-principles.md`, `coding-standards.md`, `code-review.md`, `responsive-design.md` (frontend/mobile only), `sources.md`, and the stack note itself — see "Why the global rules are duplicated per stack" below.

## How this repository is maintained

- **Every note reflects a decision I've actually made** — a missing rule is a question for me, never something to infer.
- **Frontmatter + cross-links**: each note opens with `--- tags: [...] ---`. Related notes are referenced inline, where actually relevant, via plain relative Markdown links with a display alias — `[git-conventions](00-global/git-conventions.md)` — resolved relative to the file containing the link, so an agent reading raw Markdown can follow the reference with a direct file read, no search needed.
- **Exception: `04-infra/` notes are deliberately self-contained** — no cross-links, and no naming of specific stacks (no "ASP.NET Core" or "Spring Boot" mentions). An agent working on infra shouldn't be pulled toward a particular stack or another infra piece it doesn't need.
- **New note or renamed file** → add the corresponding entry to this README's "Structure" section (and, if it's a new top-level category, describe it there too).
- **No empty scaffolding**: don't create a new folder or stub note "for later." A folder/note is added at the moment it has real content — this repository's own `coding-standards.md` rule, applied reflexively to itself.
- **Single source of truth, no duplication — except the deliberate per-stack duplication below.** Outside that one exception, a rule lives in exactly one note; if two notes need the same rule, one links to the other instead of restating it.

### Why the global rules are duplicated per stack

`architecture-principles.md`, `coding-standards.md`, `code-review.md`, and `responsive-design.md` used to live once in `00-global/` and get shared by every stack. In practice they weren't stack-agnostic — they carried code examples in one language and cross-links naming every other stack, so reading one pulled an agent's attention toward stacks that project never uses. They're now duplicated into each stack's own folder instead, adapted to that stack (native code examples, zero mentions of other stacks) — deliberately trading single-source-of-truth for isolation. A change to one of these rules has to be applied by hand to every stack folder that carries it; there is no automated sync. Keep that cost in mind before adding a new universal rule — decide once whether it's worth propagating everywhere or belongs in just the stacks it actually applies to.

### Git

This repository itself is worked directly on `main` with plain, imperative Conventional Commits (`docs: …` for nearly everything, since content here is documentation) — it does not follow the two-branch (`main`/`dev`) workflow that [git-conventions](00-global/git-conventions.md) prescribes for the projects *consuming* this repository. Match the existing commit style (`git log`) rather than that note's prescriptive policy when committing here.

## Why it's public

This lives on my GitHub so it never gets lost — it's the accumulated result of a lot of trial and error — and so anyone curious, a recruiter included, can see exactly how I think about architecture, conventions, and working with AI agents day to day.

Plain, dependency-free Markdown throughout — notes cross-link with relative Markdown links, so it reads fine directly on GitHub or in any editor, no special tooling required.
