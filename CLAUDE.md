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
  `## See also` section linking related notes via Obsidian `[[wikilinks]]`. Follow this pattern for
  new notes; update the `## See also` list (in both directions) when a note gains a new relevant
  neighbor.
- **`Index.md`** is the map of the vault — any new note or renamed file needs a corresponding entry
  there, and in `README.md`'s "What it covers" section if it's a new top-level category.
- **No empty scaffolding**: don't create a new folder or stub note "for later." A folder/note is
  added at the moment it has real content, per the vault's own rule in `coding-standards.md`
  applied reflexively to itself.
- **Single source of truth, no duplication**: a rule lives in exactly one note. If two notes seem
  to need the same rule, one should link to the other via `[[wikilink]]`, not restate it.

## Structure

- `00-global/` — rules that apply to every project regardless of stack: `coding-standards.md`,
  `architecture-principles.md`, `git-conventions.md`, `code-review.md`, `tech-stack-map.md`
  (decision map for picking a stack), `responsive-design.md`, `sources-conventions.md`,
  `readme-conventions.md`.
- `01-mobile/`, `02-web/`, `03-backend/`, `04-infra/` — one note per stack/technology, each
  resolving its own key technical decisions (state management, DI, testing stack, etc.) so an
  importing project never has to guess.
- `05-templates/` — `how-to-compose-claude-md.md` (defines the `@path` import mechanism and the
  generic pattern) plus ready-made `template-*.md` files for common stack combinations, meant to be
  copied as the starting point for a new project's `CLAUDE.md`.
- `skill/` — two Claude Code skills that automate consuming this vault from *other* project repos
  (see below). They are not consumed from inside this repo.

## The `@path` import mechanism (critical to understand before editing)

Documented in `05-templates/how-to-compose-claude-md.md`. Key points that affect how notes must be
written:

- `@path` (absolute path, forward slashes) inlines a file's full content into an importing
  project's `CLAUDE.md` at session start. Import chains go up to 4 hops; vault notes never
  `@import` each other (they use `[[wikilinks]]` only), so a project → vault note is always a
  single hop.
- `@path` only expands as plain text — **never inside a fenced code block or inline code span**.
  Any note that shows example import lines must keep them outside triple-backtick fences.
- Imported content is inlined in full, unsummarized — keep individual notes focused so importing
  projects aren't forced to pull in irrelevant bulk.

## The two `skill/` agents

`skill/new-project-claude-md` and `skill/existing-project-claude-md` are Claude Code skills
(deployed to `~/.claude/skills/`) that read *this vault* to generate or audit a `CLAUDE.md` in
some other project's repo:

- **`new-project-claude-md`**: conversational — gathers project intent, picks a stack from
  `tech-stack-map.md` and an architecture from `architecture-principles.md`, builds the import list,
  writes `CLAUDE.md` at the target project's root.
- **`existing-project-claude-md`**: evidence-based — inspects an already-started repo (dependency
  manifests, folder layout, `docker-compose.yml`) to detect stack/architecture/infra actually in
  use, then reconciles against vault defaults instead of assuming a greenfield choice.

Both hardcode this vault's absolute path in their `SKILL.md` (`C:\Users\juan.velazquez\dev-context-system`)
— **update that path in both files (and here) if the vault ever moves**, and keep the two skills'
descriptions of the vault's structure in sync with reality when the structure changes. Both skills
only ever write to the *target* project; neither edits this vault.

## Git

This repo itself is worked directly on `main` with plain, imperative Conventional Commits
(`docs: …` for nearly everything, since content here is documentation) — it does not follow the
two-branch (`main`/`dev`) workflow that `00-global/git-conventions.md` prescribes for the projects
*consuming* this vault. Match the existing commit style (`git log`) rather than the prescriptive
policy in that note when committing to this repo.
