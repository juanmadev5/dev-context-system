---
tags: [jetpack-compose, architecture]
---

# Architecture Principles — Jetpack Compose

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults. The choice depends on project size and how much business logic it carries — not on personal preference.

- **Clean Architecture** (layered: domain / data / presentation, dependencies pointing inward) — default for apps with real, long-lived business logic, or any project expected to grow, be maintained by more than one person, or live for years.
- **Vertical Slice** (organized by feature/use case, each slice owning its own request→response path) — default for medium-sized apps where features are largely independent of each other, or apps that have enough logic to not be a script but not enough to justify a full layered split.
- **Neither / plain structure** — for small, low-logic apps. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on Android framework classes, Compose, or infrastructure details. Infrastructure (Retrofit clients, Room, DataStore) implements interfaces defined by the layer/slice that needs them, not the other way around.
  ```kotlin
  // Bad — domain depends on a concrete infrastructure type
  class OrderService(private val repository: RoomOrderRepository) // concrete Room DAO wrapper

  // Good — domain depends on an abstraction it owns; infrastructure implements it
  interface OrderRepository {
      suspend fun getById(id: String): Order?
  }

  class OrderService(private val repository: OrderRepository)
  ```
- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP client, or Compose, the boundary is probably wrong.
  ```kotlin
  // Bad — the business rule can't be tested without a live data source
  class PricingService(private val dataSource: CustomerDataSource) {
      suspend fun calculateFinalPrice(customerId: String): Double {
          val customer = dataSource.fetchCustomer(customerId)
          return if (customer.isVip) customer.basePrice * 0.9 else customer.basePrice
      }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  fun calculateFinalPrice(isVip: Boolean, basePrice: Double): Double =
      if (isVip) basePrice * 0.9 else basePrice
  ```
- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies the same way under Clean Architecture and Vertical Slice — no carve-out for "it's simple" or "there's only one implementation today."
  ```kotlin
  interface EmailSender {
      suspend fun send(to: String, body: String)
  }

  class SmtpEmailSender : EmailSender {
      override suspend fun send(to: String, body: String) { /* ... */ }
  }

  class OrderConfirmationHandler(private val emailSender: EmailSender) {
      // Testable in isolation with a fake EmailSender — no real SMTP call needed
      // to verify a confirmation gets sent after an order is placed.
  }
  ```
- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one feature and Vertical Slice in another within the same codebase without a deliberate, documented reason.

## See also

- [[01-mobile/jetpack-compose/jetpack-compose|jetpack-compose]]
- [[01-mobile/jetpack-compose/coding-standards|coding-standards]]
- [[01-mobile/jetpack-compose/code-review|code-review]]
