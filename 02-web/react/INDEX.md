---
tags: [react, index]
---

# React — Index

Single entry point for a React project's `CLAUDE.md`. Importing this one file (via `@path`) pulls in every rule this stack needs — architecture, coding standards, code review, responsive design, and how to record consulted documentation — with nothing from any other stack in this vault.

A React project's `CLAUDE.md` should import this file plus [[00-global/git-conventions|git-conventions]] (genuinely stack-agnostic, stays global), [[02-web/tailwind-css|tailwind-css]] (styling, paired with virtually every React project), and whichever [[04-infra/local-infrastructure|infra notes]] the project actually pairs with (database, auth, storage, deployment).

Content imports (in order):

@architecture-principles.md
@coding-standards.md
@code-review.md
@responsive-design.md
@sources.md
@react.md
