---
tags: [react, coding-standards]
---

# Coding Standards — React

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to React and never contradicts it. [react.md](react.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to `.map`/`.filter`/`.reduce`-style chains.
- Bad (meaning only recoverable by re-reading the surrounding types):

```typescript
  const errorsByField = validationErrors.reduce((acc, e) => {
    acc[e.field] = [...(acc[e.field] ?? []), e.message];
    return acc;
  }, {} as Record<string, string[]>);
```

- Good (each name says what the item is, no guessing required):

```typescript
  const errorsByField = validationErrors.reduce((accumulated, validationError) => {
    accumulated[validationError.field] = [
      ...(accumulated[validationError.field] ?? []),
      validationError.message,
    ];
    return accumulated;
  }, {} as Record<string, string[]>);
```

## No magic values

- Closed sets of options use TypeScript `enum`/union of string literals, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Comments

- Doc-comment format: JSDoc `/** */`.
- Examples that qualify in this stack: multi-step chains such as a `.map().filter().reduce()`-style pipeline; generic/abstract code such as complex generics, higher-order components, render-prop or Strategy-style patterns.

## Error handling

See [error-handling](error-handling.md) for the API client's error normalization, the global error boundary, and structured logging.
