---
tags: [angular, documentation, sources]
---

# Sources — Angular

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `angular.dev` (Angular itself), `rxjs.dev`, `typescriptlang.org`, and each library's own docs domain listed below.

## Angular

- Official docs entry point / overview: https://angular.dev/overview
- Signals guide: https://angular.dev/guide/signals
- Standalone components (anatomy, the `imports` array): https://angular.dev/guide/components
- Reactive Forms guide: https://angular.dev/guide/forms/reactive-forms
- Dependency injection guide: https://angular.dev/guide/di
- `takeUntilDestroyed` (RxJS interop) guide: https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed

## RxJS

- Introduction / overview: https://rxjs.dev/guide/overview
- Observable guide: https://rxjs.dev/guide/observable

## TypeScript

- The TypeScript Handbook: https://www.typescriptlang.org/docs/handbook/intro.html

## Jest

- Getting started guide: https://jestjs.io/docs/getting-started

## Playwright

- Documentation homepage: https://playwright.dev/
- Installation / getting started: https://playwright.dev/docs/intro

## Transloco

- Official documentation site: https://jsverse.gitbook.io/transloco
- Quickstart / installation guide: https://jsverse.gitbook.io/transloco/getting-started/installation

## ESLint

- Getting Started guide: https://eslint.org/docs/latest/use/getting-started

## angular-eslint

- Official repository and setup instructions: https://github.com/angular-eslint/angular-eslint
- Flat config configuration guide: https://github.com/angular-eslint/angular-eslint/blob/main/docs/CONFIGURING_ESLINT.md

## If a source isn't listed here

This index covers the libraries/topics this vault's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [[02-web/angular/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

  ```markdown
  ## some-new-package

  - What the page covers: https://...
  ```

## See also

- [[02-web/angular/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
