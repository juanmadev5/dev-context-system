---
tags: [templates, claude-code, index]
---

# How to compose a project's CLAUDE.md

This vault is the **single source of truth**. Individual project repos don't duplicate these rules — their `CLAUDE.md` imports the relevant notes from here via Claude Code's `@path` import syntax, plus a short project-specific section.

## The mechanism (verified against Claude Code's current behavior)

- `@path` anywhere in a CLAUDE.md file (inline or on its own line) tells Claude Code to inline that file's content at session start.
- **Absolute paths work**: `@/home/juanma/Documents/dev-context-system/00-global/coding-standards.md` pulls that exact file in, regardless of where the project lives on disk.
- Import chains can go up to **4 hops deep**. The notes in this vault don't themselves use `@imports` (only Obsidian `[[wikilinks]]` for cross-references), so a project's CLAUDE.md → vault note is a single hop — no risk of hitting the limit.
- The **first time** a project's CLAUDE.md imports a file from outside the project directory (i.e. from this vault), Claude Code shows a one-time approval dialog. Approve it once per project; it won't ask again for that project.
- `@path` inside a fenced code block or inline code span is **not** expanded — it stays literal text. So when writing a project's CLAUDE.md, the `@import` lines must sit as plain text, not inside triple-backtick blocks.
- Imported content isn't summarized or trimmed — it loads in full. Keep the *set* of imports scoped to what that project actually uses (don't import every stack note "just in case").

## Steps to set up a new project

1. Pick the stack from [[tech-stack-map]].
2. Copy the closest matching template from this folder (`05-templates/`) — or build the import list from scratch using the pattern below.
3. Save it as `CLAUDE.md` at the root of the project repo.
4. Fill in the "Project-specific context" section at the bottom (domain, business rules unique to this project, anything not already covered by the vault).
5. Open the project in Claude Code once and approve the external-file-import prompt when it appears.
6. If this project needs to deviate from a stack note's defaults (state management, DI, testing stack, etc.), document the deviation and why in the project-specific section — or, if it's actually a durable preference you want applied to *every* future project on that stack, update the note in the vault instead, so it's captured once and reused.

## Generic pattern

``` markdown
# <Project Name>

<one-line description of what this project is>

## Context imports

@/home/juanma/Documents/dev-context-system/00-global/coding-standards.md
@/home/juanma/Documents/dev-context-system/00-global/architecture-principles.md
@/home/juanma/Documents/dev-context-system/00-global/git-conventions.md
@/home/juanma/Documents/dev-context-system/00-global/sources-conventions.md
@/home/juanma/Documents/dev-context-system/<layer>/<stack-note>.md
@/home/juanma/Documents/dev-context-system/04-infra/<relevant-infra-notes>.md

## Project-specific context

- Domain / purpose: ...
- Anything that deviates from the vault defaults, and why: ...
- Deviations from the vault's stack defaults for this project, and why (if any): ...
```

Ready-made examples for common combos:

- [[template-web-complex-angular-aspnet]]
- [[template-web-medium-vue-supabase]]
- [[template-landing-astro]]
- [[template-landing-html]]
- [[template-mobile-flutter]]
- [[template-mobile-jetpack-compose]]

## Keeping the vault and projects in sync

- This vault is edited directly — there's no build/export step. A project's CLAUDE.md always reads the live vault files at session start, so an edit here applies to every project immediately without re-copying anything.
- Only the *project-specific* section of each project's CLAUDE.md needs manual upkeep. If a rule turns out to be wrong or you change your mind, fix it in the vault note, not in every project's CLAUDE.md.

## See also

- [[tech-stack-map]]
