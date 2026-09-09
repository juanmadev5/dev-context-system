---
tags: [aspnet-core, architecture]
---

# Architecture Principles — ASP.NET Core

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults for backends with real logic, and both pair with **Controllers** — see [aspnet-core.md](aspnet-core.md)'s API style section for why. The choice between them depends on project size and how much business logic it carries — not on personal preference.

- **Clean Architecture** (layered: domain / application / infrastructure / presentation, dependencies pointing inward) — default for backends with substantial, long-lived business logic, or any project expected to grow, be maintained by more than one person, or live for years.
- **Vertical Slice Architecture** (organized by feature/use case, each slice owning its own request→response path) — default for medium-sized backends where a full layered split would add more ceremony than value but the app still has enough logic to not be a script.
- **Neither / plain structure** — for small, low-logic projects, exposed as **Minimal APIs**. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on frameworks, UI, or infrastructure details. Infrastructure (DB, HTTP clients, storage SDKs) implements interfaces defined by the layer/slice that needs them, not the other way around.

```csharp
  // Bad — domain depends on a concrete infrastructure type
  public class OrderService
  {
      private readonly SqlOrderRepository _repository; // concrete EF Core class
      public OrderService(SqlOrderRepository repository) => _repository = repository;
  }

  // Good — domain depends on an abstraction it owns; infrastructure implements it
  public interface IOrderRepository
  {
      Task<Order?> GetByIdAsync(Guid id);
  }

  public class OrderService
  {
      private readonly IOrderRepository _repository;
      public OrderService(IOrderRepository repository) => _repository = repository;
  }
```

- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or a UI framework, the boundary is probably wrong.

```csharp
  // Bad — the business rule can't be tested without a live DB
  public class PricingService
  {
      public decimal CalculateFinalPrice(Guid customerId)
      {
          var customer = _dbContext.Customers.Find(customerId);
          return customer.IsVip ? customer.BasePrice * 0.9m : customer.BasePrice;
      }
  }

  // Good — the rule is pure and testable in isolation; DB access is separate
  public static class PricingRules
  {
      public static decimal CalculateFinalPrice(bool isVip, decimal basePrice) =>
          isVip ? basePrice * 0.9m : basePrice;
  }
```

- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies the same way under Clean Architecture and Vertical Slice — no carve-out for "it's simple" or "there's only one implementation today."

```csharp
  public interface IEmailSender
  {
      Task SendAsync(string to, string body);
  }

  public class SmtpEmailSender : IEmailSender
  {
      public Task SendAsync(string to, string body) { /* ... */ }
  }

  public class OrderConfirmationHandler
  {
      private readonly IEmailSender _emailSender;
      public OrderConfirmationHandler(IEmailSender emailSender) => _emailSender = emailSender;
      // Testable in isolation with a fake IEmailSender — no real SMTP call needed
      // to verify a confirmation gets sent after an order is placed.
  }
```

- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one module and Vertical Slice in another within the same codebase without a deliberate, documented reason.

- **Wrap external systems behind a boundary**: never let an external API's, payment gateway's, or third-party SDK's own field names, shapes, or error codes leak past the layer that talks to it. Define a model this app owns and convert at the edge — if the external service changes its contract, only the conversion code changes, not every consumer across the app.

```csharp
  // Bad — the payment provider's response shape leaks into application code
  public class OrderService
  {
      public async Task ConfirmPaymentAsync(Guid orderId)
      {
          var response = await _stripeClient.GetPaymentIntentAsync(orderId.ToString());
          if (response.status == "succeeded") { /* ... */ } // provider's own field/values
      }
  }

  // Good — a boundary converts the provider's shape into a type this app owns
  public interface IPaymentGateway
  {
      Task<PaymentResult> GetPaymentResultAsync(Guid orderId);
  }

  public class StripePaymentGateway : IPaymentGateway
  {
      public async Task<PaymentResult> GetPaymentResultAsync(Guid orderId)
      {
          var response = await _stripeClient.GetPaymentIntentAsync(orderId.ToString());
          return response.status switch
          {
              "succeeded" => PaymentResult.Succeeded,
              "requires_payment_method" => PaymentResult.Failed,
              _ => PaymentResult.Pending,
          };
      }
  }
```

- **Make invalid states unrepresentable**: design types so a value that shouldn't exist can't be constructed, instead of relying on runtime checks scattered across the app. If a field is only meaningful once an order is paid, model a dedicated `PaidOrder` where that field is non-nullable, instead of a nullable field on `Order` that every consumer has to null-check and guess about.

```csharp
  // Bad — PaidAt is nullable on every Order, every consumer must guess/null-check
  public class Order
  {
      public OrderStatus Status { get; set; }
      public DateTime? PaidAt { get; set; }
  }

  // Good — a paid order is its own type where PaidAt is guaranteed to exist
  public record PendingOrder(Guid Id, IReadOnlyList<OrderItem> Items);
  public record PaidOrder(Guid Id, IReadOnlyList<OrderItem> Items, DateTime PaidAt);
```
