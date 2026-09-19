---
tags: [jetpack-compose, coding-standards]
---

# Coding Standards — Jetpack Compose

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to Jetpack Compose and never contradicts it. [jetpack-compose.md](jetpack-compose.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to `.map`/`.filter`/`.fold`-style chains.
- `it` counts as a generic placeholder when it shadows meaning — name the parameter explicitly instead.
- Bad (meaning only recoverable by re-reading the surrounding types):

```kotlin
  val errorsByField = validationErrors
      .fold(mutableMapOf<String, MutableList<String>>()) { acc, e ->
          acc.getOrPut(e.field) { mutableListOf() }.add(e.message)
          acc
      }
```

- Good (each name says what the item is, no guessing required):

```kotlin
  val errorsByField = validationErrors
      .fold(mutableMapOf<String, MutableList<String>>()) { accumulated, validationError ->
          accumulated.getOrPut(validationError.field) { mutableListOf() }.add(validationError.message)
          accumulated
      }
```

## No magic values

- Closed sets of options use Kotlin `enum class`, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Architecture

- The empty-folder rule ([engineering-principles](../../00-global/engineering-principles.md)) also applies to a feature's `domain/` — it isn't created at all until it gets its first file.

## Comments

- Doc-comment format: KDoc `/** */`.
- Examples that qualify in this stack: multi-step chains such as a `.map().fold()`-style pipeline; generic/abstract code such as complex generics, reflection-like patterns, Strategy/Visitor-style implementations.

## Error handling

- System boundaries in this stack: user input, external API responses, platform/IPC calls. Internal invariants are already enforced by Kotlin's type system or null-safety, so don't add defensive checks for them.
- See [error-handling](error-handling.md) for the global error boundary, exception mapping, and structured logging.
