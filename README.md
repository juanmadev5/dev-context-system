# Dev Context System

A personal knowledge base of engineering decisions I've already made — architecture, naming, git hygiene, per-stack conventions — so AI coding agents write code the way I actually want it written instead of improvising a fresh, "reasonable" convention on every project.

## What this is

This is **not** a software project — it's a repository of plain Markdown notes, with no application code, no dependency manifest, and no build/lint/test tooling. "Working on it" means editing notes accurately and keeping them internally consistent.

Every note reflects a decision I've actually made and stand behind — never a generic best-practices checklist copied from somewhere else. If a rule is missing, that's a question for me, not something for an agent to infer or fill in.

## How an agent should use this repository

1. **Give the agent read access to this directory.** It doesn't get imported live into a project — it's a library an agent consults once, at project-definition time.
2. **During a new project's Spec-Driven Development phase**, once the project's shape and stack are understood, use [tech-stack-map](00-global/tech-stack-map.md) to confirm (or pick) the stack.
3. **Read the layers, from most general to most specific** — each layer adds to the one before it and never contradicts it:
   1. **[00-global/](00-global/)** — the rules that apply to every stack: [engineering-principles](00-global/engineering-principles.md), [code-review](00-global/code-review.md), [testing-principles](00-global/testing-principles.md), [error-handling-principles](00-global/error-handling-principles.md), [sources-policy](00-global/sources-policy.md), [git-conventions](00-global/git-conventions.md), [dependency-management](00-global/dependency-management.md), [docker](00-global/docker.md), and, for any UI stack, [responsive-design](00-global/responsive-design.md).
   2. **The stack's own folder** — list it (e.g. `ls 01-mobile/flutter/`) and read it in this order: `architecture-principles.md`, `coding-standards.md`, `code-review.md`, `testing.md`, `error-handling.md`, `sources.md`, then the stack's own note (e.g. `flutter.md`).
   3. **The shared notes the project actually uses**:
      - [tailwind-css](02-web/tailwind-css.md) — every web project.
      - [rest-api-design](03-backend/rest-api-design.md) — every backend project that exposes a REST API.
4. **Write the project's own conventions from what was read** — its `/docs/` folder and a `CLAUDE.md` that points at `/docs/` with plain relative links, never back at this repository. Then **verify against the source notes more than once** that no rule was dropped, weakened, or contradicted along the way. From that point the project is self-contained: no live dependency on this repo, no cross-repo imports. If a rule here changes later, redo the affected part when it's relevant to do so.

This deliberately replaces an earlier version of this system that used Claude Code's `@path` import syntax to pull content live into a project's `CLAUDE.md`. That tied every consuming project to one tool's import mechanism; plain files in `/docs/` work with any agent that can read a directory.

## Structure

- `00-global/` — the notes that stay genuinely stack-agnostic, written once and shared by every stack: [engineering-principles](00-global/engineering-principles.md), [code-review](00-global/code-review.md), [testing-principles](00-global/testing-principles.md), [error-handling-principles](00-global/error-handling-principles.md), [responsive-design](00-global/responsive-design.md), [sources-policy](00-global/sources-policy.md), [git-conventions](00-global/git-conventions.md), [dependency-management](00-global/dependency-management.md), [docker](00-global/docker.md), [readme-conventions](00-global/readme-conventions.md), and [tech-stack-map](00-global/tech-stack-map.md) (the decision map, read once per project).
- `01-mobile/` — [flutter](01-mobile/flutter/flutter.md), [jetpack-compose](01-mobile/jetpack-compose/jetpack-compose.md).
- `02-web/` — [angular](02-web/angular/angular.md), [vuejs](02-web/vuejs/vuejs.md), [react](02-web/react/react.md), plus the shared [tailwind-css](02-web/tailwind-css.md) (not a stack of its own — read alongside whichever web stack is chosen).
- `03-backend/` — [aspnet-core](03-backend/aspnet-core/aspnet-core.md), [spring-boot](03-backend/spring-boot/spring-boot.md), plus the shared [rest-api-design](03-backend/rest-api-design.md) (not a stack of its own — read alongside whichever backend stack exposes a REST API).

Each mobile/web/backend stack folder bundles only what's specific to that stack — its own `architecture-principles.md`, `coding-standards.md`, `code-review.md`, `testing.md`, `error-handling.md`, `sources.md`, and the stack note itself. The stack files build on the global notes above (code examples, tooling, doc-comment format, mocking mechanics) and link up to them; see "Why the rules are layered" below.

## How this repository is maintained

- **Every note reflects a decision I've actually made** — a missing rule is a question for me, never something to infer.
- **Frontmatter + cross-links**: each note opens with `--- tags: [...] ---`. Related notes are referenced inline, where actually relevant, via plain relative Markdown links with a display alias — `[git-conventions](00-global/git-conventions.md)` — resolved relative to the file containing the link, so an agent reading raw Markdown can follow the reference with a direct file read, no search needed.
- **New note or renamed file** → add the corresponding entry to this README's "Structure" section (and, if it's a new top-level category, describe it there too).
- **No empty scaffolding**: don't create a new folder or stub note "for later." A folder/note is added at the moment it has real content — the rule in [engineering-principles](00-global/engineering-principles.md), applied reflexively to itself.
- **Single source of truth, no duplication.** A rule lives in exactly one note; if two notes need the same rule, one links to the other instead of restating it. Global notes never name a stack and carry no language-specific code; stack notes link *up* to the global ones, never sideways to another stack.

### Why the rules are layered

An earlier version copied the whole rule set into every stack folder so an agent working on one stack never read another one's code examples or cross-links. That kept stacks isolated but meant a change to one rule had to be applied by hand to seven files. Now the stack-agnostic prose (naming, DRY/SOLID, comments, definition of done, review process, testing structure, error-handling policy, responsive principle) lives once in `00-global/` — with no code examples and no stack names, so it can't pull an agent toward a stack the project doesn't use — and each stack folder holds only what's specific to it. The agent reads the global layer plus its own stack folder and nothing else, so isolation is preserved without the duplication.

What's still stack-scoped on purpose: `architecture-principles.md` (each stack's layout and code examples in its own language) and the link lists in each `sources.md`.

### Git

This repository itself is worked directly on `main` with plain, imperative Conventional Commits (`docs: …` for nearly everything, since content here is documentation) — it does not follow the two-branch (`main`/`dev`) workflow that [git-conventions](00-global/git-conventions.md) prescribes for the projects *consuming* this repository. Match the existing commit style (`git log`) rather than that note's prescriptive policy when committing here.

## Why it's public

This lives on my GitHub so it never gets lost — it's the accumulated result of a lot of trial and error — and so anyone curious, a recruiter included, can see exactly how I think about architecture, conventions, and working with AI agents day to day.

Plain, dependency-free Markdown throughout — notes cross-link with relative Markdown links, so it reads fine directly on GitHub or in any editor, no special tooling required.
