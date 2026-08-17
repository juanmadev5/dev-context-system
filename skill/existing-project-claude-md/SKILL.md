---
name: existing-project-claude-md
description: Generates or audits a CLAUDE.md for a project that already has code, by inspecting the repo (stack, folder structure, infra in use) rather than starting from a blank conversation, then reconciling findings against the juanma dev-context-system vault. Use when the user wants to document, onboard, or set up a CLAUDE.md for an existing/already-started project, or asks to check whether a project's current CLAUDE.md is still accurate.
---

# Existing project CLAUDE.md composer / auditor

Companion to `new-project-claude-md`, for the opposite situation: the project already has code and
already made decisions — some of which may deliberately diverge from the
`dev-context-system` vault's defaults for good reasons. This skill's job is to **document reality
first**, then reconcile it against the vault — never to silently rewrite the codebase's existing
conventions to match the vault, and never to overwrite a hand-authored `CLAUDE.md` without showing
what changed.

**Vault path (this machine, hardcoded — update if the vault ever moves):**
```
C:/Users/juan.velazquez/OneDrive - Cabaña los Troncos S.A/Escritorio/notas/dev-context-system
```

Read `Index.md` and `05-templates/how-to-compose-claude-md.md` in that vault first if unfamiliar
with the `@path` import mechanism this skill relies on.

## Where the output goes

`CLAUDE.md` at the root of the project currently being worked in (the current working directory).
Never in the vault.

## Flow

### 1. Detect the stack from evidence, not conversation

Inspect the repo for concrete signals before asking anything:

- `package.json` dependencies → Angular (`@angular/core`), React (`react`), Vue (`vue`), Astro
  (`astro`).
- `*.csproj` / `*.sln` → ASP.NET Core (`03-backend/aspnet-core.md`).
- `pom.xml` / `build.gradle` (non-Android) → Spring Boot (`03-backend/spring-boot.md`).
- `pubspec.yaml` → Flutter (`01-mobile/flutter.md`).
- `build.gradle.kts` with Compose dependencies → Jetpack Compose
  (`01-mobile/jetpack-compose.md`).
- Plain `index.html` + Tailwind config, no framework → `02-web/html-tailwind.md`.
- Match every detected stack signal to its vault note. If a signal doesn't cleanly match anything
  in the vault (an unfamiliar framework, a stack not covered), say so explicitly instead of
  guessing the closest note — that's a case to ask the user about, not infer.

### 2. Detect the architecture actually in use

Read the folder structure, don't assume:

- `domain/`, `application/`, `infrastructure/`, `presentation/` (or equivalent layered names) →
  Clean Architecture.
- `features/<name>/` or `modules/<name>/` each owning its own slice → Vertical Slice.
- No consistent layering → no formal pattern (fine for small/low-logic projects per
  `00-global/architecture-principles.md`).

This step is observation only — record what's actually there, not what the vault would have
recommended for a greenfield project of this shape. Compare against
`00-global/architecture-principles.md`'s size/complexity criteria only in step 5, to flag
divergence, never to justify silently changing the code.

### 3. Detect infrastructure in use

`docker-compose.yml` services, config/env references to Keycloak, PostgreSQL, Redis, AWS S3,
Supabase, deployment config (CI files, Dockerfiles). Match each to its `04-infra/` note. Only
import what's actually wired up — a commented-out or unused service in `docker-compose.yml`
doesn't count.

### 4. Fill the gaps that inspection can't answer

Read the existing README (if any) for domain/purpose. For anything still missing — business
rules, why a particular deviation from a vault default exists, what's actually planned vs. legacy
— ask the user rather than guessing (per the vault's own "Ask vs. assume" rule in
`00-global/coding-standards.md`). Keep this short: only ask what steps 1-3 genuinely couldn't
determine from the repo itself.

### 5. Reconcile against the vault — flag, don't force

For each place where the detected reality diverges from what the vault would default to (e.g. the
code uses Vertical Slice but `architecture-principles.md`'s criteria would suggest Clean
Architecture for a project this size/complexity), surface it as a question to the user:

- "Keep this as a documented, deliberate deviation in the project-specific section?", or
- "Is this actually something we should bring in line with the vault default?"

Never resolve this silently in either direction. The generated `CLAUDE.md` reflects whichever
answer the user gives, with the reason recorded in "Project-specific context" (per
`how-to-compose-claude-md.md`'s deviation-documentation pattern).

### 6a. If no `CLAUDE.md` exists yet

Write one: imports for every stack/infra note matched in steps 1-3, plus
`00-global/coding-standards.md`, `00-global/architecture-principles.md`,
`00-global/git-conventions.md`, `00-global/code-review.md`, `00-global/sources-conventions.md`
(the always-on set), plus `00-global/responsive-design.md` / `00-global/readme-conventions.md`
where relevant. "Project-specific context" section filled from steps 4-5.

### 6b. If a `CLAUDE.md` already exists

Audit it, don't replace it outright:

- Check every `@path` import actually resolves to a file that exists in the vault — flag any that
  don't (renamed/moved note, typo).
- Check the documented stack/architecture/infra against what steps 1-3 actually found in the
  code — flag drift (documented but not detected, or detected but not documented).
- Check whether "Project-specific context" still reflects the current state of the project, or
  describes something that's since changed.
- Present findings as a proposed diff and confirm with the user before writing — an existing
  hand-authored `CLAUDE.md` may encode decisions (deviations, exceptions) that aren't visible from
  the code alone, so treat it the way any existing file with unclear prior intent is treated:
  investigate before overwriting.

### 7. Hand off

If this is the first time this project's `CLAUDE.md` imports from the vault, tell the user:
Claude Code will show a one-time approval prompt for external-file imports on next load — approve
it once, it won't ask again for this project.

## Hard rules

- Never import a vault note that doesn't exist — verify first.
- Never infer the stack, architecture, or infra by guessing when the repo doesn't clearly show
  it — ask.
- Never silently rewrite the project's actual code/structure to match a vault default just because
  this skill noticed a divergence — that's a separate, explicit decision the user makes, if ever.
- Never overwrite an existing `CLAUDE.md` without first showing what would change and getting
  confirmation.
- This skill only writes the target project's `CLAUDE.md`. It never edits anything inside the
  vault itself.
