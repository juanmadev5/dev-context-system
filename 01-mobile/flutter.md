---
tags: [mobile, flutter, dart]
---

# Flutter

Cross-platform mobile. Default choice for mobile projects that need to ship on both iOS and Android from one codebase — see [[tech-stack-map]]. Use [[jetpack-compose]] instead when the project is Android-only and needs deep platform integration.

## Project structure

Follow [[architecture-principles]]: Clean Architecture for apps with real business logic, Vertical Slice / simple feature-first structure for smaller apps. A Clean Architecture layout in Dart typically looks like:

```
lib/
  core/               # shared utilities, constants, error types, DI setup
  features/
    <feature_name>/
      domain/         # entities, repository interfaces, use cases — no Flutter/Dart-package deps
      data/           # repository implementations, DTOs/models, data sources (remote/local)
      presentation/   # widgets, screens, state management for this feature
```

## Conventions

- File names: `snake_case.dart`. Classes: `PascalCase`. Members/variables: `camelCase`. Constants: `camelCase` (or `SCREAMING_SNAKE_CASE` only if the team already uses that style consistently) — never inline literals, per [[coding-standards]].
- Widgets: prefer small, composable widgets over large `build()` methods. Extract a widget when a subtree has its own responsibility or is reused.
- Null-safety is mandatory; avoid `!` (force-unwrap) except where nullability has already been exhaustively proven earlier in the same scope.
- Enums: use Dart `enum`s for closed sets of values (never raw strings/ints), and when serializing use `.name`, never `.index` — see [[coding-standards]].

## Styling

- No Tailwind here (mobile, not web). Use a centralized theme (`ThemeData`, design tokens as constants) instead of hardcoded colors/spacing scattered across widgets.

## State management

- **Bloc** (`flutter_bloc`) for screens/features with genuinely complex state (multiple events, transitions, side effects worth modeling explicitly).
- **Cubit** for simple state (a handful of straightforward state changes, no need for full event-driven modeling). Don't reach for full Bloc when a Cubit says the same thing with less ceremony.
- Both live in the `presentation/` layer of a feature — see [[architecture-principles]]. States and events are modeled as sealed classes/unions, never raw strings/bools/enum-index checks, per [[coding-standards]].

## Dependency injection

- **GetIt** as the service locator, registered at app startup (per-feature registration modules to keep `main.dart`/DI setup from becoming a dumping ground).

## Navigation

- **go_router** for all routing, including deep links and nested navigation.

## Testing

- `flutter_test` + **mocktail** for unit/widget tests. Since state management is Bloc/Cubit, pair with `bloc_test` for testing state transitions instead of driving them manually through mocktail alone.

## Internationalization (i18n)

- **Before starting a new Flutter project (or a significant new app within one), the agent must explicitly ask whether the app needs to support multiple languages.** Never assume either way — don't silently build it in "just in case", and don't silently hardcode strings assuming it'll never be needed.
- **If yes**: use Flutter's official stack — `flutter_localizations` + `intl`, with ARB files (`lib/l10n/*.arb`) as the source of truth, code-generated via `flutter gen-l10n`. Route all user-facing strings through it from the start, even for a single-language initial release — retrofitting i18n later means touching every widget again.
- **If no**: plain UI strings are fine (still the one exception to English-only in [[coding-standards]]), but keep them out of widget bodies — a single `strings.dart` (or similar) constants file avoids the same string being duplicated across widgets, per DRY.

## Responsiveness

- Use `LayoutBuilder` / `MediaQuery` (or a breakpoints abstraction built on top of them) to adapt layout between phone and tablet form factors and to handle orientation changes — never hardcode pixel/logical-pixel dimensions that assume one specific device. See [[responsive-design]] for the full cross-platform principle.

## Logging

- **`logger`** package for all logging — leveled output (debug/info/warning/error) with readable formatting, instead of scattering `print()`/`debugPrint()` calls through the codebase.

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `flutter analyze` (or `dart analyze`) — must pass clean, no new errors/warnings/lints introduced by the change.

## See also

- [[coding-standards]], [[architecture-principles]], [[responsive-design]]
- [[jetpack-compose]]
