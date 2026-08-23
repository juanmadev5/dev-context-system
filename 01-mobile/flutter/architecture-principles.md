---
tags: [flutter, architecture]
---

# Architecture Principles — Flutter

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults. The choice depends on project size and how much business logic it carries — not on personal preference.

- **Clean Architecture** (layered: domain / data / presentation, dependencies pointing inward) — default for apps with real, long-lived business logic, or any project expected to grow, be maintained by more than one person, or live for years.
- **Vertical Slice** (organized by feature/use case, each slice owning its own request→response path) — default for medium-sized apps where features are largely independent of each other, or apps that have enough logic to not be a script but not enough to justify a full layered split.
- **Neither / plain structure** — for small, low-logic apps. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on Flutter widgets, packages, or infrastructure details. Infrastructure (HTTP clients, local storage, platform channels) implements interfaces defined by the layer/slice that needs them, not the other way around.
  ```dart
  // Bad — domain depends on a concrete infrastructure type
  class OrderService {
    final SqlOrderRepository repository; // concrete implementation
    OrderService(this.repository);
  }

  // Good — domain depends on an abstraction it owns; infrastructure implements it
  abstract class OrderRepository {
    Future<Order?> getById(String id);
  }

  class OrderService {
    final OrderRepository repository;
    OrderService(this.repository);
  }
  ```
- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or the widget tree, the boundary is probably wrong.
  ```dart
  // Bad — the business rule can't be tested without a live data source
  class PricingService {
    double calculateFinalPrice(String customerId) {
      final customer = dataSource.fetchCustomer(customerId);
      return customer.isVip ? customer.basePrice * 0.9 : customer.basePrice;
    }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  double calculateFinalPrice({required bool isVip, required double basePrice}) =>
      isVip ? basePrice * 0.9 : basePrice;
  ```
- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies the same way under Clean Architecture and Vertical Slice — no carve-out for "it's simple" or "there's only one implementation today."
  ```dart
  abstract class EmailSender {
    Future<void> send(String to, String body);
  }

  class SmtpEmailSender implements EmailSender {
    Future<void> send(String to, String body) async { /* ... */ }
  }

  class OrderConfirmationHandler {
    final EmailSender emailSender;
    OrderConfirmationHandler(this.emailSender);
    // Testable in isolation with a fake EmailSender — no real SMTP call needed
    // to verify a confirmation gets sent after an order is placed.
  }
  ```
- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one feature and Vertical Slice in another within the same codebase without a deliberate, documented reason.

## See also

- [[01-mobile/flutter/flutter|flutter]]
- [[01-mobile/flutter/coding-standards|coding-standards]]
- [[01-mobile/flutter/code-review|code-review]]
