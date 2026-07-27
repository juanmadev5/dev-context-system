---
tags: [web, angular, typescript]
---

# Angular

Default for complex web apps with heavy business logic — see [[tech-stack-map]]. Pairs with **Clean Architecture** (see [[architecture-principles]]): Angular's DI system and module boundaries map naturally onto layered architecture.

## Project structure

```
src/app/
  core/                 # singletons: auth, http interceptors, app-wide services, DI tokens
  shared/               # reusable, stateless UI (components, pipes, directives) with no feature-specific logic
  features/
    <feature>/
      domain/            # models, business rules, interfaces — framework-agnostic where possible
      data/              # services implementing domain interfaces (HTTP, storage)
      presentation/       # components, containers, feature routing
```

- Standalone components by default (no `NgModule` boilerplate unless the project predates it or has a specific reason to keep modules).
- Smart/container components (feature-level, own data-fetching) vs. dumb/presentational components (inputs/outputs only) — keep the split explicit.

## Conventions

- File names: `kebab-case`, Angular's standard suffixes (`.component.ts`, `.service.ts`, `.guard.ts`, etc.).
- Strong typing everywhere — avoid `any`; prefer `unknown` + narrowing when the type genuinely isn't known yet.
- Reactive state (RxJS `Observable`s / signals) over imperative subscription juggling; always unsubscribe or use `async` pipe / `takeUntilDestroyed()`.
- Constants and enums per [[coding-standards]] — no magic strings for route paths, HTTP header names, storage keys, etc.
- **Package manager: pnpm**, never `npm`/`yarn` — see [[tech-stack-map]].

## Styling

- [[tailwind-css]] — no separate component-scoped CSS files unless Tailwind genuinely can't express something (rare).

## State management

- **Signals-only**: `signal`/`computed`/`effect` exposed from injectable services for shared/global state, component-local `signal`s for local state. No NgRx — it adds more ceremony than these projects need. If a specific feature's state truly becomes Redux-shaped (many interdependent actions, need for time-travel debugging), that's a deliberate, documented exception, not the default.

## HTTP & error handling

- A central `HttpInterceptor` catches HTTP errors and normalizes them into a common error type (e.g. `AppError`) before they reach feature services/components. Features consume the normalized type, never raw `HttpErrorResponse` deep in component code.

## Forms

- **Reactive Forms** with custom validators written as plain functions (`ValidatorFn`/`AsyncValidatorFn`), reused across forms via [[coding-standards]]'s DRY rule. No third-party form libraries unless a project has a specific need (e.g. many backend-driven dynamic forms) that justifies the added dependency.

## Testing

- **Jest** for unit tests, **Playwright** for e2e — not the Angular CLI's Jasmine/Karma/Cypress defaults.

## Internationalization (i18n)

- **Before starting a new Angular project (or a significant new app within one), the agent must explicitly ask whether the app needs to support multiple languages.** Same rule as [[flutter]] — never assume either way.
- **If yes**: use **Transloco**, not Angular's built-in extraction-based i18n (which requires a separate build per locale) and not ngx-translate. Translation keys live in per-locale JSON files, lazy-loaded per feature module where the project is large enough for that to matter. No literal UI strings in templates — every user-facing string goes through a translation key from the start, even for a single-language initial release.
- **If no**: UI strings stay hardcoded (the one exception to English-only in [[coding-standards]]), but centralized — a shared constants file (or one per feature, per [[architecture-principles]]'s layout) instead of the same literal duplicated across components, per DRY.

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `tsc --noEmit` — type-checking. `ng build` also catches type errors, but `tsc --noEmit` is faster when a full build isn't needed.
- `eslint .` (via `ng lint` if `@angular-eslint` is set up in the project) — must pass clean.

## See also

- [[coding-standards]], [[architecture-principles]], [[tailwind-css]], [[responsive-design]]
- [[aspnet-core]], [[spring-boot]] (typical backend pairing)
