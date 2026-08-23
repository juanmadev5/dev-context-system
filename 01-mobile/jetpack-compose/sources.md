---
tags: [jetpack-compose, documentation, sources]
---

# Sources — Jetpack Compose

Every time an agent consults external documentation while working on a Jetpack Compose project — official docs, API references, package documentation — the source gets recorded in that project's `docs/SOURCES.md`. This is what lets a human (or another agent) later verify *where* a convention or implementation detail actually came from, instead of trusting it blindly or re-deriving it from scratch.

## When to record

- Any time a web lookup (fetching a page, searching, reading a library's official docs) actually informs a decision, an implementation detail, or code written during a task — record it.
- A lookup that turns out irrelevant to what got written doesn't need an entry — this is about traceability for what actually shaped the work, not a log of every request made.
- `docs/SOURCES.md` is created the first time it's actually needed, per [[01-mobile/jetpack-compose/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation — don't scaffold it empty upfront in a new project.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search. If the official docs genuinely don't cover something, that's worth flagging to the developer rather than filling the gap from an unofficial source.
- For this stack, official means: `developer.android.com` (Jetpack Compose, Android platform APIs) and `kotlinlang.org` (Kotlin language, coroutines, standard library).

## Format

`docs/SOURCES.md`, grouped by library under a `##` heading, one bullet per source:

```markdown
## Navigation Compose

- Nested graphs: https://developer.android.com/guide/navigation/design/nested-graphs

## Coroutines

- StateFlow vs SharedFlow: https://kotlinlang.org/docs/flow.html#stateflow-and-sharedflow
```

- Group by the library the source belongs to, not by date or by task — the file accumulates over the project's lifetime as a reference index, not a session log.
- One bullet per distinct source; don't duplicate an entry that's already there for the same URL.
- The bullet's label is a short, specific description of what the link actually covers, never just "docs" or the bare URL with no label.

## See also

- [[01-mobile/jetpack-compose/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
