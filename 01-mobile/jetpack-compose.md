---
tags: [mobile, android, jetpack-compose, kotlin]
---

# Android Jetpack Compose

Android-native UI toolkit. Use instead of [[flutter]] when the project is Android-only and benefits from deep platform integration — see [[tech-stack-map]].

## Architecture pattern: MVVM

The `presentation/` layer always follows **MVVM**, regardless of project size:

- **View** — the Composable. Pure function of state, no business logic, no direct data access. Renders UI state and forwards user events upward.
- **ViewModel** — one per screen (or per cohesive feature). Owns the UI state, exposes it as a single immutable `StateFlow`/`State` (never multiple loosely-related mutable fields), and handles the events the View forwards to it.
- **Model** — the `domain`/`data` layers below: use cases, repositories, entities. The ViewModel talks to these, never the View directly.

This is not optional per-project — it's the default for every Compose project, on top of whichever of [[architecture-principles]]'s two structures (Clean Architecture / Vertical Slice) the project's size calls for.

## Project structure

A Clean Architecture layout on Android typically looks like:

```
app/
  core/                 # shared utilities, DI modules, constants, error types
  feature/<feature>/
    domain/             # use cases, repository interfaces, models — pure Kotlin
    data/                # repository implementations, DTOs, remote/local data sources
    presentation/        # Composables (View), ViewModels, UI state — MVVM as above
```

- Composables are pure functions of state where possible: state flows down, events flow up to the ViewModel.

## Conventions

- File/class names: `PascalCase.kt`. Functions/properties: `camelCase`. Composable functions: `PascalCase` (they're treated as UI declarations, not regular functions).
- Constants: `companion object` `const val`, never inline literals — see [[coding-standards]].
- Enums: Kotlin `enum class` for closed sets; when serializing use the enum's `.name`, never `.ordinal` — see [[coding-standards]].
- Avoid business logic inside Composables — keep them declarative; logic lives in the ViewModel/domain layer.

## Styling

- No Tailwind here (native Android, not web). Use Material 3 theming (`MaterialTheme`, design tokens as constants) instead of hardcoded colors in Composables.

## No hardcoded strings or dimensions

Strict rule, no exceptions: a Composable never contains a literal UI string or a literal dimension value.

- **All user-facing strings** go in `res/values/strings.xml`, referenced via `stringResource(R.string.xxx)` — never a string literal inside a Composable's `Text()`, `contentDescription`, etc. This also carries the [[flutter]]-equivalent benefit here: since strings already live in `strings.xml`, adding i18n later (`values-es/strings.xml`, etc.) is just adding resource files, not a rewrite — so still confirm with the user upfront whether multi-language support is needed, same as [[flutter]]'s i18n rule, since that affects how `strings.xml` is organized (keys, pluralization via `plurals`, etc.) from the start.
- **All dimensions** (padding, spacing, sizes, corner radii, elevations) go in `res/values/dimens.xml`, referenced via `dimensionResource(R.dimen.xxx)` — never a literal `.dp`/`.sp` value inline in a Composable.
- This is a stricter, Android-native alternative to "constants in code" for these two categories specifically — it's not optional even though [[coding-standards]]'s general no-magic-values rule would technically be satisfied by a Kotlin `const val` instead. Use the XML resources.

## Dependency injection

- **Hilt** for all DI (modules scoped per feature where it makes sense, app-level singletons in a `core`/`di` module).

## Navigation

- **Navigation Compose** (Jetpack's official navigation library) for all routing.

## Async / reactive

- Kotlin **Coroutines + Flow** cover this on their own — `StateFlow` for UI state exposed from ViewModels, `Flow` for data streams from repositories. No RxJS/RxJava unless interoperating with legacy Rx-based code.

## Testing

- **JUnit** for logic/ViewModel tests, **Compose UI Testing** for Composables, **Turbine** for asserting on `Flow` emissions instead of manual `collect` boilerplate in tests.

## Responsiveness

- Use Material 3's `WindowSizeClass` (adaptive layouts) and `BoxWithConstraints` to adapt layout across Android's phone/tablet/foldable range — never a fixed `dp` layout that assumes one screen size. See [[responsive-design]] for the full cross-platform principle.

## Logging

- **Timber** for all logging, not the raw `android.util.Log` — it drops the manual tag boilerplate (auto-derives the tag from the calling class) and makes it trivial to plant a no-op tree in release builds instead of shipping debug logs to production.

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `./gradlew lint` — Android Lint, catches Android-specific issues (resource misuse, manifest problems, performance/API-level warnings).
- `./gradlew detekt` — Kotlin static analysis (complexity, style, common bug patterns), the de facto standard for Kotlin projects. Both must pass clean.

## See also

- [[coding-standards]], [[architecture-principles]], [[responsive-design]]
- [[flutter]]
