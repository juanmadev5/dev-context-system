---
tags: [flutter, coding-standards]
---

# Coding Standards — Flutter

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to Flutter and never contradicts it. [flutter.md](flutter.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to `.map`/`.where`/`.fold`-style chains.
- Bad (meaning only recoverable by re-reading the surrounding types):

```dart
  final errorsByField = validationErrors
      .fold<Map<String, List<String>>>({}, (acc, e) {
    acc.putIfAbsent(e.field, () => []).add(e.message);
    return acc;
  });
```

- Good (each name says what the item is, no guessing required):

```dart
  final errorsByField = validationErrors
      .fold<Map<String, List<String>>>({}, (accumulated, validationError) {
    accumulated.putIfAbsent(validationError.field, () => []).add(validationError.message);
    return accumulated;
  });
```

## No magic values

- Closed sets of options use Dart `enum`, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Architecture

- The empty-folder rule ([engineering-principles](../../00-global/engineering-principles.md)) also applies to a feature's `domain/use_cases/` — it isn't created at all until it gets its first file.

## Comments

- Doc-comment format: `///`.
- Examples that qualify in this stack: multi-step chains such as a `.map().fold()`-style pipeline; generic/abstract code such as complex generics, reflection-like patterns, Strategy/Visitor-style widgets.

## Error handling

See [error-handling](error-handling.md) for the global error boundary, exception mapping, and structured logging.
