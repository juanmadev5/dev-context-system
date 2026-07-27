---
tags: [global, documentation, sources]
---

# Sources Conventions

Every time an agent consults external documentation while working on a project — official docs, API references, RFCs/specs — the source gets recorded in that project's `docs/SOURCES.md`. This is what lets a human (or another agent) later verify *where* a convention or implementation detail actually came from, instead of trusting it blindly or re-deriving it from scratch.

## When to record

- Any time a web lookup (fetching a page, searching, reading a library's official docs) actually informs a decision, an implementation detail, or code written during a task — record it.
- A lookup that turns out irrelevant to what got written doesn't need an entry — this is about traceability for what actually shaped the work, not a log of every request made.
- `docs/SOURCES.md` is created the first time it's actually needed, per [[coding-standards]]'s "no empty folders" reasoning applied to documentation — don't scaffold it empty upfront in a new project.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the project's own repo (README, wiki, official guide), or a relevant standard/spec (an RFC, a W3C spec, etc.).
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search. If the official docs genuinely don't cover something, that's worth flagging to the developer rather than filling the gap from an unofficial source.
- Examples of "official" for stacks already in this vault: `learn.microsoft.com` (.NET, ASP.NET Core, EF Core), `docs.spring.io` (Spring Boot), `kotlinlang.org` (Kotlin), `developer.android.com` (Jetpack Compose), `docs.flutter.dev`, `react.dev`, `vuejs.org`, `angular.dev`, `docs.astro.build`, `tailwindcss.com/docs`, `www.postgresql.org/docs`, `www.keycloak.org/documentation`, `supabase.com/docs`.

## Format

`docs/SOURCES.md`, grouped by stack/library under a `##` heading, one bullet per source:

```markdown
## ASP.NET Core

- EF Core: https://learn.microsoft.com/en-us/ef/core/
- Minimal APIs: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

## PostgreSQL

- CHECK constraints: https://www.postgresql.org/docs/current/ddl-constraints.html
```

- Group by the stack/library the source belongs to, not by date or by task — the file accumulates over the project's lifetime as a reference index, not a session log.
- One bullet per distinct source; don't duplicate an entry that's already there for the same URL.
- The bullet's label is a short, specific description of what the link actually covers (`EF Core`, `Minimal APIs`, `CHECK constraints`) — never just "docs" or the bare URL with no label.

## See also

- [[coding-standards]], [[readme-conventions]]
