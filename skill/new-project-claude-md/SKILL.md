---
name: new-project-claude-md
description: Composes a new project's CLAUDE.md through conversation — asks about the project, picks stack and architecture from the juanma dev-context-system vault, and fills in project-specific context. Use when the user wants to start a new project, asks to generate/create a CLAUDE.md, or asks what stack to use for something they're about to build.
---

# New project CLAUDE.md composer

Turns a conversation about a project idea into that project's `CLAUDE.md`, sourced from the
`dev-context-system` vault — the single source of truth for this developer's coding standards,
architecture rules, and stack conventions. Never invent rules here; every rule the generated
`CLAUDE.md` relies on must already live in the vault as an importable note.

**Vault path (this machine, hardcoded — update if the vault ever moves):**
```
C:\Users\juan.velazquez\dev-context-system
```

Read `Index.md` and `05-templates/how-to-compose-claude-md.md` in that vault first if unfamiliar
with its structure — they define the `@path` import mechanism and the generic pattern this skill
automates.

## Where the output goes

The generated `CLAUDE.md` is written to the root of the **project the user is currently working
in** (the current working directory when this skill runs) — never into the vault itself. If the
current directory doesn't look like the intended project root (e.g. it's empty, or clearly the
wrong place), confirm the target path with the user before writing.

## Conversation flow

### 1. Gather the essentials

Ask (don't assume — see the vault's own "Ask vs. assume" rule in `00-global/coding-standards.md`,
it applies here too):

- What is this project? One or two sentences: domain, purpose, who uses it.
- What kind of app: mobile, web, backend API, landing page, or a combination?
- Any stack preference already decided, or should it be recommended based on the above?
- Any known business rules, constraints, or integrations (auth provider, existing backend to talk
  to, data model quirks) worth capturing now?

Keep this a real conversation, not a form — ask follow-ups where the answer is vague enough that
picking a stack or architecture off it would be a guess.

### 2. Resolve the stack

- If the user already named a stack, confirm it's one covered by the vault (`01-mobile/`,
  `02-web/`, `03-backend/`, `04-infra/`) before proceeding.
- If not, read `00-global/tech-stack-map.md` and recommend a stack based on what was described in
  step 1. Present the recommendation and the reasoning briefly; get confirmation before moving on
  — don't silently commit to a stack the user didn't sign off on.

### 3. Resolve the architecture

Read `00-global/architecture-principles.md`. Apply its decision criteria (project size, how much
business logic it carries, expected lifespan/team size) to what was described in step 1: Clean
Architecture, Vertical Slice, or no formal pattern for a small/low-logic project. State the pick
and the one-line reason.

### 4. Check for an existing template

Look in `05-templates/` for a `template-*.md` that already matches the chosen stack + architecture
combination. If one matches, use it as the base for the import list and folder-structure
guidance instead of building from scratch.

### 5. Build the import list (if no template matches, or as a base to adjust)

Follow the generic pattern in `05-templates/how-to-compose-claude-md.md`:

- Always: `00-global/coding-standards.md`, `00-global/architecture-principles.md`,
  `00-global/git-conventions.md`, `00-global/code-review.md`, `00-global/sources-conventions.md`.
- The chosen stack note(s) from `01-mobile/`, `02-web/`, `03-backend/`.
- Only the `04-infra/` notes the project actually uses (auth provider, storage, DB, deployment
  target) — never import an infra note "just in case it comes up later."
- `00-global/responsive-design.md` and `00-global/readme-conventions.md` when relevant (most web
  and mobile projects; skip for a pure backend API with no UI).

**Before writing any `@path` line, verify the target file actually exists in the vault** — never
reference a note by a guessed or remembered name without checking. Use the vault's absolute path
from above, forward slashes, one import per line, as plain text (not inside a code fence — `@path`
only expands outside fenced/inline code, per `how-to-compose-claude-md.md`).

### 6. Fill in project-specific context

Using what came out of step 1 (and any deviation decided in steps 2-3 that the user explicitly
chose over the vault default), write the "Project-specific context" section: domain/purpose,
business rules unique to this project, and any deviation from vault defaults with the reason —
per the vault's own rule, a deviation only belongs in the vault itself if it's a durable
preference for *future* projects too, not just this one.

### 7. Write the file and hand off

- Write `CLAUDE.md` at the project root, following the generic pattern's structure (title,
  one-line description, `## Context imports`, `## Project-specific context`).
- Tell the user: the first time this `CLAUDE.md` loads in Claude Code, it'll show a one-time
  approval prompt for importing files from outside the project directory (the vault) — approve it
  once, it won't ask again for this project.
- If the user wants to deviate from a vault default currently, still write the deviation in the
  project-specific section — don't edit the vault from inside this flow. Editing the vault is a
  separate, deliberate action the user takes when a project-specific deviation turns out to be a
  standing preference.

## Hard rules

- Never import a vault note that doesn't exist — check first.
- Never guess the stack or architecture without either an explicit user answer or an explicit,
  stated recommendation the user confirmed.
- Don't over-import: only the notes the project actually needs, not the whole vault.
- This skill only writes the target project's `CLAUDE.md`. It never edits anything inside the
  vault itself.
