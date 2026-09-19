---
tags: [angular, coding-standards]
---

# Coding Standards — Angular

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to Angular and never contradicts it. [angular.md](angular.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to `.filter`/`.map`/`.reduce`-style chains and RxJS operators (`.pipe(map(...), filter(...))`).
- Bad (meaning only recoverable by re-reading the surrounding types):

```typescript
  const errorsByField = validationErrors
    .filter((e) => e.severity === 'error')
    .reduce<Record<string, string[]>>((acc, e) => {
      (acc[e.field] ??= []).push(e.message);
      return acc;
    }, {});
```

- Good (each name says what the item is, no guessing required):

```typescript
  const errorsByField = validationErrors
    .filter((validationError) => validationError.severity === 'error')
    .reduce<Record<string, string[]>>((accumulated, validationError) => {
      (accumulated[validationError.field] ??= []).push(validationError.message);
      return accumulated;
    }, {});
```

## No magic values

- Closed sets of options use TypeScript `enum` (or a `const`-object union), per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Comments

- Doc-comment format: JSDoc `/** */`.
- Examples that qualify in this stack: multi-step chains such as a `.pipe(switchMap(...), map(...), catchError(...))` pipeline; generic/abstract code such as complex generics, decorator-based metaprogramming, Strategy/Visitor-style services.

## Error handling

- System boundaries in this stack: user input, external API responses, browser APIs. Internal invariants are already enforced by TypeScript's type system, so don't add defensive checks for them.
- See [error-handling](error-handling.md) for the HTTP interceptor's error normalization, the global error handler, and structured logging.
