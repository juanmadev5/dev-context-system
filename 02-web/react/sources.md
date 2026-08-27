---
tags: [react, documentation, sources]
---

# Sources — React

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `react.dev` (React itself), `vite.dev`, `typescriptlang.org`, and each library's own docs domain listed below.

## React

- Official docs entry point (Learn / Quick Start): https://react.dev/learn
- Built-in Hooks reference: https://react.dev/reference/react/hooks

## Vite

- Getting Started guide (scaffolding, TypeScript templates): https://vite.dev/guide/

## TypeScript

- The TypeScript Handbook: https://www.typescriptlang.org/docs/handbook/intro.html

## Zustand

- Official documentation site: https://zustand.docs.pmnd.rs/
- Introduction / getting started: https://zustand.docs.pmnd.rs/learn/getting-started/introduction

## React Router

- Official documentation homepage: https://reactrouter.com/
- Routing configuration (Framework Mode): https://reactrouter.com/start/framework/routing
- Data loading with loaders (for centralizing auth/permission guards): https://reactrouter.com/start/framework/data-loading

## React Hook Form

- Get Started guide (installation, validation, TypeScript, schema validation): https://react-hook-form.com/get-started

## Zod

- Documentation homepage: https://zod.dev/
- API reference (schema definitions): https://zod.dev/api

## Vitest

- Documentation homepage: https://vitest.dev/
- Getting Started guide: https://vitest.dev/guide/

## React Testing Library

- Introduction / docs: https://testing-library.com/docs/react-testing-library/intro/

## Playwright

- Documentation homepage: https://playwright.dev/
- Installation / getting started: https://playwright.dev/docs/intro

## react-i18next

- Documentation homepage: https://react.i18next.com/
- Quick Start guide: https://react.i18next.com/guides/quick-start

## ESLint

- Getting Started guide: https://eslint.org/docs/latest/use/getting-started

## eslint-plugin-react-hooks

- Official plugin source and README (in the React monorepo, maintained by the React core team): https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks

## If a source isn't listed here

This index covers the libraries/topics this vault's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [[02-web/react/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

  ```markdown
  ## some-new-package

  - What the page covers: https://...
  ```

## See also

- [[02-web/react/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
