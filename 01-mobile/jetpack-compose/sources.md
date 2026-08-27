---
tags: [jetpack-compose, documentation, sources]
---

# Sources — Jetpack Compose

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `developer.android.com` (Jetpack Compose, Android platform APIs) and `kotlinlang.org` (Kotlin language, coroutines, standard library) — plus a library's own repo README when it has no separate docs site (e.g. Timber, Turbine).

## Android app architecture

- Guide to app architecture (overview, UDF, layers): https://developer.android.com/topic/architecture
- UI layer guide (UI state, state holders, UDF): https://developer.android.com/topic/architecture/ui-layer
- Data layer guide (repositories, data sources): https://developer.android.com/topic/architecture/data-layer

## Jetpack Compose

- Get started with Jetpack Compose: https://developer.android.com/develop/ui/compose/documentation
- State and Jetpack Compose (state hoisting, `remember`, `rememberSaveable`): https://developer.android.com/develop/ui/compose/state
- Test your Compose layout (Compose UI Testing APIs): https://developer.android.com/develop/ui/compose/testing

## Hilt

- Dependency injection with Hilt: https://developer.android.com/training/dependency-injection/hilt-android

## Navigation 3

- Navigation 3 overview: https://developer.android.com/guide/navigation/navigation-3
- Understand and implement the basics (back stack, keys, `NavDisplay`): https://developer.android.com/guide/navigation/navigation-3/basics

## Kotlin Coroutines & Flow

- Coroutines guide: https://kotlinlang.org/docs/coroutines-guide.html
- Asynchronous Flow — StateFlow and SharedFlow: https://kotlinlang.org/docs/coroutines-flow.html

## Room

- Save data in a local database using Room: https://developer.android.com/training/data-storage/room

## DataStore

- DataStore guide (Preferences/Proto DataStore): https://developer.android.com/topic/libraries/architecture/datastore

## Material 3 WindowSizeClass

- Use window size classes (adaptive layouts): https://developer.android.com/develop/ui/compose/layouts/adaptive/window-size-classes

## Timber

- Official README (JakeWharton/timber): https://github.com/JakeWharton/timber

## Turbine

- Official README (cashapp/turbine): https://github.com/cashapp/turbine

## detekt

- Official docs site: https://detekt.dev/

## Android Lint

- Improve your code with lint checks: https://developer.android.com/studio/write/lint

## If a source isn't listed here

This index covers the libraries/topics this vault's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [[01-mobile/jetpack-compose/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

  ```markdown
  ## some_new_library

  - What the page covers: https://...
  ```

## See also

- [[01-mobile/jetpack-compose/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
