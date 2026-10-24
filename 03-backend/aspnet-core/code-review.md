---
tags: [aspnet-core, code-review]
---

# Code Review — ASP.NET Core

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to ASP.NET Core and never contradicts it. The stack's static-analysis commands are in [aspnet-core.md](aspnet-core.md)'s `## Static analysis` section.

## Performance

- N+1 queries, unnecessary loops/allocations, unpaginated large payloads.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Injection (SQL, NoSQL, command, LDAP) — unparameterized queries, string-concatenated commands.
- Broken access control — missing or incorrect authorization checks per endpoint/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded or logged.
- CSRF/CORS misconfiguration, where applicable.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

```csharp
  // Bad — EF Core entity returned straight from the API
  [HttpGet("{id}")]
  public async Task<Customer> Get(Guid id) => await _dbContext.Customers.FindAsync(id);

  // Good — a DTO shaped for the API contract, decoupled from the persistence model
  [HttpGet("{id}")]
  public async Task<CustomerResponse> Get(Guid id)
  {
      var customer = await _dbContext.Customers.FindAsync(id);
      return new CustomerResponse(customer.Id, customer.Name, customer.Email);
  }
```

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```csharp
  // Bad — reordering or inserting a member silently changes stored meaning
  public enum OrderStatus { Pending, Paid, Shipped }
  context.SaveJson(new { status = (int)order.Status });

  // Good — stable regardless of member order
  context.SaveJson(new { status = order.Status.ToString() });
```
