---
tags: [flutter, documentation, sources]
---

# Sources — Flutter

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs.

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `docs.flutter.dev` (Flutter itself), `dart.dev` (language/core libraries), `pub.dev` (a package's own page), and `bloclibrary.dev` (flutter_bloc's own official docs site).

## Flutter

- Official app architecture guide (MVVM, dependency injection, design patterns): https://docs.flutter.dev/app-architecture
- State management overview and options: https://docs.flutter.dev/data-and-backend/state-mgmt/options
- Testing overview (unit, widget, integration tests): https://docs.flutter.dev/testing/overview
- Internationalization guide (ARB files, `flutter gen-l10n`): https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization

## Dart

- Language tour / core language features: https://dart.dev/language

## flutter_bloc

- Official pub.dev package page: https://pub.dev/packages/flutter_bloc
- Core Bloc/Cubit concepts (BlocBuilder, BlocProvider, BlocSelector): https://bloclibrary.dev/bloc-concepts/
- Flutter-specific Bloc concepts and widgets: https://bloclibrary.dev/flutter-bloc-concepts/
- Architecture guidance, including Bloc-to-Bloc communication: https://bloclibrary.dev/architecture/

## go_router

- Official pub.dev package page: https://pub.dev/packages/go_router
- Nested navigation via `StatefulShellRoute` (tabs with independent nav stacks): https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute-class.html
- Deep linking: https://pub.dev/documentation/go_router/latest/topics/Deep%20linking-topic.html
- Redirection / route guards: https://pub.dev/documentation/go_router/latest/topics/Redirection-topic.html

## GetIt

- Official pub.dev package page: https://pub.dev/packages/get_it

## mocktail

- Official pub.dev package page: https://pub.dev/packages/mocktail

## bloc_test

- Official pub.dev package page: https://pub.dev/packages/bloc_test

## logger

- Official pub.dev package page: https://pub.dev/packages/logger

## If a source isn't listed here

This index covers the libraries/topics this repository's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [coding-standards](coding-standards.md)'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

  ```markdown
  ## some_new_package

  - What the page covers: https://...
  ```

