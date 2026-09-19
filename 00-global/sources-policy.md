---
tags: [global, documentation, sources]
---

# Sources Policy

Which documentation an agent may rely on, for every stack. Each stack's `sources.md` is a curated index of that stack's official documentation and says which domains count as official for it; the rules below apply to all of them.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.

## If a source isn't listed in the stack's `sources.md`

The index covers the libraries/topics this repository's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed there:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [engineering-principles](engineering-principles.md)'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

```markdown
## some-new-package

- What the page covers: https://...
```
