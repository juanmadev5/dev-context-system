---
tags: [vuejs, documentation, sources]
---

# Sources — Vue.js

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `vuejs.org` (Vue itself), `vite.dev`, and each library's own docs domain listed below.

## Vue.js

- Official docs entry point / introduction: <https://vuejs.org/guide/introduction.html>
- Composition API FAQ (what/why, relation to the Options API): <https://vuejs.org/guide/extras/composition-api-faq.html>
- `<script setup>` SFC syntax reference: <https://vuejs.org/api/sfc-script-setup.html>
- Using Vue with TypeScript: <https://vuejs.org/guide/typescript/overview.html>

## Vite

- Getting Started guide: <https://vite.dev/guide/>

## Pinia

- Official documentation homepage: <https://pinia.vuejs.org/>

## Vue Router

- Official documentation homepage: <https://router.vuejs.org/>
- Navigation Guards guide: <https://router.vuejs.org/guide/advanced/navigation-guards.html>
- Route Meta Fields guide: <https://router.vuejs.org/guide/advanced/meta.html>

## Vitest

- Getting Started guide: <https://vitest.dev/guide/>

## Vue Test Utils

- Official documentation homepage: <https://test-utils.vuejs.org/>

## MSW (Mock Service Worker)

- Official documentation home: <https://mswjs.io/>

## loglevel

- Official README (pimterry/loglevel): <https://github.com/pimterry/loglevel>

## Playwright

- Installation / getting started: <https://playwright.dev/docs/intro>

## vue-i18n

- Introduction: <https://vue-i18n.intlify.dev/guide/introduction>
- Getting Started guide: <https://vue-i18n.intlify.dev/guide/essentials/started>
- Composition API usage guide: <https://vue-i18n.intlify.dev/guide/advanced/composition>

## vue-tsc

- Official README (usage, `--noEmit`, requirements) in the `vuejs/language-tools` repo: <https://github.com/vuejs/language-tools/blob/master/packages/tsc/README.md>

## eslint-plugin-vue

- Introduction: <https://eslint.vuejs.org/>
- User Guide (installation/configuration): <https://eslint.vuejs.org/user-guide/>

## If a source isn't listed here

This index covers the libraries/topics this repository's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [coding-standards](coding-standards.md)'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

```markdown
  ## some-new-package

  - What the page covers: https://...
```
