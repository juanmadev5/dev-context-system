---
tags: [spring-boot, coding-standards]
---

# Coding Standards — Spring Boot

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to Spring Boot and never contradicts it. [spring-boot.md](spring-boot.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to stream chains (`.map`/`.filter`/`.collect`/`Collectors.groupingBy`-style pipelines).
- Bad (meaning only recoverable by re-reading the surrounding types):

```java
  var errors = validationErrors.stream()
      .collect(Collectors.groupingBy(
          e -> e.getField(),
          Collectors.mapping(e -> e.getMessage(), Collectors.toList())));
```

- Good (each name says what the item is, no guessing required):

```java
  var errorsByField = validationErrors.stream()
      .collect(Collectors.groupingBy(
          validationError -> validationError.getField(),
          Collectors.mapping(validationError -> validationError.getMessage(), Collectors.toList())));
```

## Control flow

- Prefer **guard clauses** over nested conditionals: check failure/exit conditions first and return or throw immediately, so the method body isn't wrapped in an ever-deepening `if`. The main logic should read top-to-bottom with no more than one level of nesting for the happy path.

```java
  // Bad — main logic buried inside nested conditions
  public Order processOrder(Order order) {
      if (order != null) {
          if (!order.getItems().isEmpty()) {
              if (order.getStatus() == OrderStatus.PENDING) {
                  return repository.save(order);
              }
          }
      }
      throw new IllegalStateException("Cannot process order");
  }

  // Good — guard clauses exit early, main logic is flat and visible
  public Order processOrder(Order order) {
      if (order == null) throw new IllegalArgumentException("order must not be null");
      if (order.getItems().isEmpty()) throw new IllegalStateException("Order has no items");
      if (order.getStatus() != OrderStatus.PENDING) throw new IllegalStateException("Order is not pending");

      return repository.save(order);
  }
```

## No magic values

- Closed sets of options use `enum`, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Architecture

- The empty-folder rule ([engineering-principles](../../00-global/engineering-principles.md)) covers packages and modules too, not just folders — a `domain`/`application`/`infrastructure`/`api` module or package is created only when it gets its first class.

## Comments

- Doc-comment format: `/** ... */` Javadoc.
- Examples that qualify in this stack: multi-step chains such as a `.stream().map().collect()`-style pipeline; generic/abstract code such as complex generics, reflection, Strategy/Visitor-style patterns.

## Error handling

See [error-handling](error-handling.md) for the global exception-handling mechanism, structured logging, and correlation ID propagation.
