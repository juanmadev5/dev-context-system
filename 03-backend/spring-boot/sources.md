---
tags: [spring-boot, documentation, sources]
---

# Sources — Spring Boot

Every time an agent consults external documentation while working on a Spring Boot project — official docs, API references, package documentation — the source gets recorded in that project's `docs/SOURCES.md`. This is what lets a human (or another agent) later verify *where* a convention or implementation detail actually came from, instead of trusting it blindly or re-deriving it from scratch.

## When to record

- Any time a web lookup (fetching a page, searching, reading a library's official docs) actually informs a decision, an implementation detail, or code written during a task — record it.
- A lookup that turns out irrelevant to what got written doesn't need an entry — this is about traceability for what actually shaped the work, not a log of every request made.
- `docs/SOURCES.md` is created the first time it's actually needed, per [[03-backend/spring-boot/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation — don't scaffold it empty upfront in a new project.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the project's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search. If the official docs genuinely don't cover something, that's worth flagging to the developer rather than filling the gap from an unofficial source.
- For this stack, official means: `docs.spring.io` (Spring Boot, Spring Framework, Spring Data, Spring Security) and `hibernate.org` for the ORM-specific behavior underneath Spring Data JPA.

## Format

`docs/SOURCES.md`, grouped by library under a `##` heading, one bullet per source:

```markdown
## Flyway

- Versioned migration naming: https://docs.spring.io/spring-boot/reference/howto/data-initialization.html#howto.data-initialization.migration-tool.flyway

## springdoc-openapi

- Documenting error responses per endpoint: https://springdoc.org/#how-can-i-add-descriptions-and-examples-to-request-response-body
```

- Group by the library the source belongs to, not by date or by task — the file accumulates over the project's lifetime as a reference index, not a session log.
- One bullet per distinct source; don't duplicate an entry that's already there for the same URL.
- The bullet's label is a short, specific description of what the link actually covers, never just "docs" or the bare URL with no label.

## See also

- [[03-backend/spring-boot/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
