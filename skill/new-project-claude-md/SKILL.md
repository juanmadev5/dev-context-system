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

Ask (don't assume — see the "Ask vs. assume" rule every stack's coding-standards note carries, it
applies here too):

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

Read the chosen stack's own `architecture-principles.md` (e.g. `02-web/react/architecture-principles.md`
once the stack is picked — every stack folder has one). Apply its decision criteria (project size,
how much business logic it carries, expected lifespan/team size) to what was described in step 1:
Clean Architecture, Vertical Slice, or no formal pattern for a small/low-logic project. State the
pick and the one-line reason.

### 4. Check for an existing template

Look in `05-templates/` for a `template-*.md` that already matches the chosen stack + architecture
combination. If one matches, use it as the base for the import list and folder-structure
guidance instead of building from scratch.

### 5. Build the import list (if no template matches, or as a base to adjust)

Follow the generic pattern in `05-templates/how-to-compose-claude-md.md`:

- Always: `00-global/git-conventions.md`.
- The chosen stack's `INDEX.md` from `01-mobile/`, `02-web/`, or `03-backend/` (e.g.
  `02-web/react/INDEX.md`) — this one file already brings in that stack's architecture,
  coding-standards, code-review, responsive-design (if applicable), and sources notes, so don't
  also import those separately.
- `02-web/tailwind-css.md` for any web stack (it's shared, not part of a stack's own `INDEX.md`).
- Only the `04-infra/` notes the project actually uses (auth provider, storage, DB, deployment
  target) — never import an infra note "just in case it comes up later."
- `00-global/readme-conventions.md` when relevant (most projects; skip only if the project
  genuinely won't have a README worth the convention, which is rare).

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

### 7. Write the file

- Write `CLAUDE.md` at the project root, following the generic pattern's structure (title,
  one-line description, `## Context imports`, `## Project-specific context`).
- If the user wants to deviate from a vault default currently, still write the deviation in the
  project-specific section — don't edit the vault from inside this flow. Editing the vault is a
  separate, deliberate action the user takes when a project-specific deviation turns out to be a
  standing preference.

### 8. Set up the commit-time static-analysis hook

The vault's rules only take effect if the agent actually reads and follows them — nothing
mechanically verifies that. Close that gap for the one thing that *is* mechanically checkable:
whether the project's static analysis command(s) pass before a commit is allowed to happen.

- For each stack chosen in step 2, read that stack's own `## Static analysis` section (e.g.
  `02-web/react/react.md`) and take only the commands marked mandatory there — skip anything
  listed as optional/"for deeper checks" (SonarAnalyzer, SpotBugs, etc.). Chain a stack's own
  mandatory commands with `&&` (all must pass).
- **Single-service project** (one stack, code at the repo root): write
  `.claude/hooks/pre-commit-static-analysis.sh` that unconditionally runs that stack's mandatory
  command(s) and exits non-zero on failure.
- **Multi-service project** (e.g. a web stack + a backend, each in its own top-level folder per
  the chosen template): the script only runs a stack's check when a file under that stack's own
  folder is actually staged — read staged paths with `git diff --cached --name-only`, and only run
  a service's command block when a staged path starts with that service's folder. Never run every
  service's check on every commit regardless of what changed. Pattern:

  ```bash
  #!/usr/bin/env bash
  staged=$(git diff --cached --name-only)
  errors=""

  if echo "$staged" | grep -q '^<service-folder>/'; then
    output=$(cd <service-folder> && <that stack's mandatory command(s)> 2>&1)
    if [ $? -ne 0 ]; then
      errors="${errors}## <service-folder> (<command>)\n${output}\n\n"
    fi
  fi
  # ...repeat the block above per service folder in this project...

  if [ -n "$errors" ]; then
    jq -n --arg reason "$errors" '{hookSpecificOutput:{hookEventName:"PreToolUse","permissionDecision":"deny","permissionDecisionReason":$reason}}'
  else
    exit 0
  fi
  ```
- Wire it into `.claude/settings.json` (project-level, so it's team-wide and gets committed) as a
  `PreToolUse` hook on the `Bash` matcher, filtered with `"if": "Bash(git commit *)"` so it only
  fires on an actual commit attempt, not on every shell command:

  ```json
  {
    "hooks": {
      "PreToolUse": [
        {
          "matcher": "Bash",
          "hooks": [
            {
              "type": "command",
              "if": "Bash(git commit *)",
              "command": "bash .claude/hooks/pre-commit-static-analysis.sh",
              "timeout": 120
            }
          ]
        }
      ]
    }
  }
  ```
- Follow the `update-config` skill's own hook-construction workflow to do this safely: read any
  existing `.claude/settings.json` first and merge (never overwrite existing hooks/permissions),
  pipe-test the script directly before wiring it into settings, validate the written JSON with
  `jq -e`, and prove the hook actually fires before considering this step done.
- Tell the user this hook now blocks `git commit` in this project until the touched stack's static
  analysis passes, and that the check that failed is what Claude sees when the commit is denied —
  so a future agent working here can read the failure and fix it before retrying, without the user
  needing to intervene.

### 9. Hand off

- Tell the user: the first time this `CLAUDE.md` loads in Claude Code, it'll show a one-time
  approval prompt for importing files from outside the project directory (the vault) — approve it
  once, it won't ask again for this project.

## Hard rules

- Never import a vault note that doesn't exist — check first.
- Never guess the stack or architecture without either an explicit user answer or an explicit,
  stated recommendation the user confirmed.
- Don't over-import: only the notes the project actually needs, not the whole vault.
- This skill only writes the target project's `CLAUDE.md` (and, per step 8, that project's
  `.claude/settings.json`/`.claude/hooks/`). It never edits anything inside the vault itself.
- The commit-time hook only ever uses commands already marked mandatory in a stack's own
  `## Static analysis` section — never invent a check that isn't already documented there.
