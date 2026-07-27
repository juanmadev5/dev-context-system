---
tags: [global, git]
---

# Git & PR Conventions

## Repository setup

- **Never assume a project should become a git repository, or that a remote should be added.** If the project isn't a git repo yet, ask the developer whether they want one initialized — and, separately, whether to add a remote — before doing either. Don't do it proactively just because work is starting.
- If the developer wants to handle repo/remote setup themselves, take their word for it and don't bring it up again — they'll ask when they're ready.

## Commits

- Write commit messages in English, imperative mood ("add", "fix", "refactor" — not "added"/"adding").
- Prefer Conventional Commits style prefixes when the project doesn't already have its own convention: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `build`, `ci`.
- Subject line: what changed. Body (only when non-obvious): why — the motivation, constraint, or incident that drove the change, not a restatement of the diff.
- **Atomic commits**: one commit, one concern. Never bundle unrelated changes into the same commit — if a commit's message needs "and" to describe it, it's two commits.
- **Only commit when explicitly asked, every time.** A previous approval doesn't carry over to the next change — no committing proactively "since it was probably wanted."
- **Never include `Co-Authored-By` (or any AI-attribution trailer) in commit messages.** The commit is authored under the user's own identity, full stop.
- **Always check `.gitignore` — and what's actually staged — before committing.** Run `git status` first and confirm nothing unintended is about to be included: build output, `.env`/secrets, editor/IDE folders, dependency directories (`node_modules`, `bin`/`obj`, etc.), local config. If `.gitignore` is missing an entry for something that shouldn't be tracked, fix `.gitignore` before staging, not after.
- **`.env` (and any local secrets file) is always listed in `.gitignore`, no exceptions** — even when its current contents look harmless (e.g. a non-secret API base URL). The rule is about the file's role, not what happens to be in it today. Only a `*.example` template with placeholder values gets committed.
- **Keep `.gitignore` proactively up to date, not just checked at commit time.** The moment a new tool, build step, or file type is introduced to the project (a new framework's build output dir, a new local config/cache folder, a new `.env` variant, an IDE adding its own folder, etc.), add the corresponding entry to `.gitignore` right then — don't wait until something unwanted actually shows up in `git status` to react to it.
- Never use destructive git operations (`--force`, `reset --hard`, amending pushed commits) without explicit confirmation.
- Never skip hooks (`--no-verify`) to work around a failing check — fix the root cause.

## Branching

- Every project has two permanent branches: **`main`** (production-ready, deployable at all times) and **`dev`** (integration branch — where work lands before it's ready for `main`).
- All work branches off `dev`, never off `main`: `feature/<short-description>`, `fix/<short-description>`, `chore/<short-description>`.
- Work branches merge back into `dev` via PR. `dev` merges into `main` only when it represents a release/deployable state — never commit or merge directly into `main`.
- Hotfixes that can't wait for the normal `dev` → `main` flow are the only exception, and still go through a PR into `main` (then get merged back into `dev` immediately after, so the fix isn't lost on the next release).

## Pull Requests

- PR title: short, imperative, under ~70 characters.
- PR description covers: what changed and why (not a line-by-line diff narration), plus a test plan / how it was verified.
- Keep PRs scoped to one concern. Split unrelated changes into separate PRs even if they were developed together.
- Before opening a PR: make sure linting, type-checking, and the test suite pass locally.

## Code review

- Review for correctness first, then architecture/consistency with existing patterns, then style.
- Flag magic values, duplicated logic, and architecture-boundary violations (see [[coding-standards]], [[architecture-principles]]) as blocking; pure style nits as non-blocking suggestions.

## See also

- [[coding-standards]]
