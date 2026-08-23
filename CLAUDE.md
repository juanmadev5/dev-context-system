# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not** a software project — it's an Obsidian vault of plain Markdown notes: the single
source of truth for how this developer wants AI coding agents to write code across *all* of their
projects (architecture, naming, git hygiene, per-stack conventions). Other projects' `CLAUDE.md`
files pull rules from here at session start via Claude Code's `@path` import syntax, so a change
made in this vault applies to every project that imports it — no build, export, or sync step.

There is no application code, no dependency manifest, and no build/lint/test tooling in this repo.
"Development" here means editing Markdown notes accurately and keeping them internally consistent.

## Working in this vault

- **Every note reflects a decision the developer has actually made and stands behind** — never add
  a "generic best practice" that isn't already implied by existing notes. If a rule is missing,
  that's a question for the developer, not something to infer or fill in.
- **Frontmatter + cross-links**: each note opens with `--- tags: [...] ---` and ends with a
  `## See also` section linking related notes via Obsidian wikilinks written with the full
  relative path and a display alias — `[[00-global/git-conventions|git-conventions]]`, not
  `[[git-conventions]]`. This keeps Obsidian's graph view/backlinks working while letting an
  agent reading the raw Markdown resolve the reference with a direct `Read`, no search needed.
  Follow this pattern for new notes; update the `## See also` list (in both directions) when a
  note gains a new relevant neighbor.
- **`Index.md`** is the map of the vault — any new note or renamed file needs a corresponding entry
  there, and in `README.md`'s "What it covers" section if it's a new top-level category.
- **No empty scaffolding**: don't create a new folder or stub note "for later." A folder/note is
  added at the moment it has real content, per the vault's own rule in `coding-standards.md`
  applied reflexively to itself.
- **Single source of truth, no duplication — except the deliberate per-stack duplication described
  below.** Outside of that one exception, a rule lives in exactly one note; if two notes seem to
  need the same rule, one should link to the other via `[[wikilink]]`, not restate it.

## Structure

- `00-global/` — the notes that stay genuinely stack-agnostic at the process level: `git-conventions.md`,
  `readme-conventions.md`, `tech-stack-map.md` (decision map for picking a stack; used once at
  setup time, not imported into a project's ongoing `CLAUDE.md`).
- `01-mobile/`, `02-web/`, `03-backend/` — one **folder per stack** (e.g. `01-mobile/flutter/`),
  each containing that stack's own `INDEX.md` plus its own copies of `architecture-principles.md`,
  `coding-standards.md`, `code-review.md`, `responsive-design.md` (frontend/mobile stacks only),
  `sources.md`, and the stack note itself (e.g. `flutter.md`). These are duplicated **on purpose**
  per stack — see "Why the global rules are duplicated per stack" below — with code examples
  rewritten in that stack's own language and every cross-stack mention stripped out. `02-web/`
  also keeps `tailwind-css.md` as a single shared, un-duplicated note (styling used by every web
  stack, never used standalone).
- `04-infra/` — one note per infrastructure piece (database, cache, auth provider, storage,
  deployment). These are **not** restructured into folders: infra is always paired with a stack
  that's already been chosen, never used standalone, so a project imports these directly alongside
  its stack's `INDEX.md`.
- `05-templates/` — `how-to-compose-claude-md.md` (defines the `@path` import mechanism and the
  generic pattern) plus ready-made `template-*.md` files for common stack combinations, meant to be
  copied as the starting point for a new project's `CLAUDE.md`.

## Why the global rules are duplicated per stack

`architecture-principles.md`, `coding-standards.md`, `code-review.md`, and `responsive-design.md`
used to live once in `00-global/` and get imported by every project. In practice they weren't
stack-agnostic — they carried code examples in one language (C#) and wikilinks naming every other
stack in the vault, so importing them pulled an agent's attention toward stacks that project never
uses. They're now duplicated into each stack's own folder instead, adapted to that stack (native
code examples, zero mentions of other stacks) — deliberately trading single-source-of-truth for
isolation. This means a change to one of these rules has to be applied by hand to every stack folder
that carries it; there is no automated sync. Keep that cost in mind before adding a new universal
rule — decide once whether it's worth propagating everywhere or belongs in just the stacks it
actually applies to.

## The `@path` import mechanism (critical to understand before editing)

Documented in `05-templates/how-to-compose-claude-md.md`. Key points that affect how notes must be
written:

- `@path` (absolute path, forward slashes) inlines a file's full content into an importing
  project's `CLAUDE.md` at session start. Import chains go up to 4 hops.
- Vault notes never `@import` each other, with one deliberate exception: each stack folder's
  `INDEX.md` uses `@path` to pull in that stack's own architecture/coding-standards/code-review/
  (responsive-design)/sources notes plus the stack note itself. Every other cross-reference in the
  vault stays a `[[wikilink]]`. A project's CLAUDE.md → stack `INDEX.md` → that stack's own notes
  is 2 hops, well under the limit.
- `@path` only expands as plain text — **never inside a fenced code block or inline code span**.
  Any note that shows example import lines must keep them outside triple-backtick fences.
- Imported content is inlined in full, unsummarized — keep individual notes focused so importing
  projects aren't forced to pull in irrelevant bulk.

## Git

This repo itself is worked directly on `main` with plain, imperative Conventional Commits
(`docs: …` for nearly everything, since content here is documentation) — it does not follow the
two-branch (`main`/`dev`) workflow that `00-global/git-conventions.md` prescribes for the projects
*consuming* this vault. Match the existing commit style (`git log`) rather than the prescriptive
policy in that note when committing to this repo.
