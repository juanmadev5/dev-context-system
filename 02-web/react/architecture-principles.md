---
tags: [react, architecture]
---

# Architecture Principles — React

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults. The choice depends on project size and how much business logic it carries — not on personal preference.

- **Clean Architecture** (layered: domain / application / infrastructure / presentation, dependencies pointing inward) — default for apps with complex business logic and long-lived state, or any project expected to grow, be maintained by more than one person, or live for years.
- **Vertical Slice** (organized by feature/use case, each slice owning its own request→response path) — default for medium-sized apps where features are largely independent of each other. This is React's most common fit — see [react.md](react.md)'s project structure.
- **Neither / plain structure** — for small, low-logic apps. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on React, the DOM, or infrastructure details. Infrastructure (HTTP clients, browser storage, third-party SDKs) implements interfaces defined by the layer/slice that needs them, not the other way around.
  ```typescript
  // Bad — the hook depends on a concrete infrastructure type
  class FetchOrderRepository {
    async getById(id: string): Promise<Order | null> { /* ... */ }
  }

  function useOrder(id: string) {
    const repository = new FetchOrderRepository(); // concrete implementation
    // ...
  }

  // Good — the hook depends on an abstraction it owns; infrastructure implements it
  interface OrderRepository {
    getById(id: string): Promise<Order | null>;
  }

  function useOrder(id: string, repository: OrderRepository) {
    // ...
  }
  ```
- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a server or rendering a component, the boundary is probably wrong.
  ```typescript
  // Bad — the business rule can't be tested without a live data source
  function usePricing(customerId: string) {
    const customer = fetchCustomer(customerId);
    return customer.isVip ? customer.basePrice * 0.9 : customer.basePrice;
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  function calculateFinalPrice(isVip: boolean, basePrice: number): number {
    return isVip ? basePrice * 0.9 : basePrice;
  }
  ```
- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies the same way under Clean Architecture and Vertical Slice — no carve-out for "it's simple" or "there's only one implementation today."
  ```typescript
  interface EmailSender {
    send(to: string, body: string): Promise<void>;
  }

  const smtpEmailSender: EmailSender = {
    send: async (to, body) => { /* ... */ },
  };

  function useOrderConfirmation(emailSender: EmailSender) {
    // Testable in isolation by passing a fake EmailSender — no real SMTP call
    // needed to verify a confirmation gets sent after an order is placed.
  }
  ```
- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one feature and Vertical Slice in another within the same codebase without a deliberate, documented reason.

