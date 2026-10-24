---
tags: [aspnet-core, coding-standards]
---

# Coding Standards — ASP.NET Core

Stack-specific additions to [engineering-principles](../../00-global/engineering-principles.md) — read that first; this note only adds what's specific to ASP.NET Core and never contradicts it. [aspnet-core.md](aspnet-core.md) adds on top of both.

## Naming

- The callback/lambda-parameter rule applies here to `.Select`/`.Where`/`.GroupBy`-style LINQ chains.
- Bad (meaning only recoverable by re-reading the surrounding types):

```csharp
  var errors = validationException.Errors
      .GroupBy(e => e.PropertyName)
      .ToDictionary(g => g.Key, g => g.Select(e => e.ErrorMessage).ToArray());
```

- Good (each name says what the item is, no guessing required):

```csharp
  var errors = validationException.Errors
      .GroupBy(validationError => validationError.PropertyName)
      .ToDictionary(
          propertyErrors => propertyErrors.Key,
          propertyErrors => propertyErrors.Select(validationError => validationError.ErrorMessage).ToArray());
```

## Control flow

- Prefer **guard clauses** over nested conditionals: check failure/exit conditions first and return or throw immediately, so the method body isn't wrapped in an ever-deepening `if`. The main logic should read top-to-bottom with no more than one level of nesting for the happy path.

```csharp
  // Bad — main logic buried inside nested conditions
  public async Task<Order> ProcessOrderAsync(Order? order)
  {
      if (order != null)
      {
          if (order.Items.Count > 0)
          {
              if (order.Status == OrderStatus.Pending)
              {
                  return await _repository.SaveAsync(order);
              }
          }
      }
      throw new InvalidOperationException("Cannot process order");
  }

  // Good — guard clauses exit early, main logic is flat and visible
  public async Task<Order> ProcessOrderAsync(Order? order)
  {
      if (order is null) throw new ArgumentNullException(nameof(order));
      if (order.Items.Count == 0) throw new InvalidOperationException("Order has no items");
      if (order.Status != OrderStatus.Pending) throw new InvalidOperationException("Order is not pending");

      return await _repository.SaveAsync(order);
  }
```

## No magic values

- Closed sets of options use `enum`, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values rule.

## Comments

- Doc-comment format: `///`.
- Examples that qualify in this stack: multi-step chains such as a `.Select().GroupBy().Aggregate()`-style pipeline; generic/abstract code such as nested generics, reflection, expression trees, Strategy/Visitor-style patterns.

## Error handling

See [error-handling](error-handling.md) for the global exception-handling mechanism, structured logging, and correlation ID propagation.
