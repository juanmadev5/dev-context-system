---
tags: [angular, architecture]
---

# Architecture Principles — Angular

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults. The choice depends on project size and how much business logic it carries — not on personal preference.

- **Clean Architecture** (layered: domain / application / infrastructure / presentation, dependencies pointing inward) — default for apps with complex business logic and long-lived state, or any project expected to grow, be maintained by more than one person, or live for years. This is the default pairing for Angular — its DI system and module boundaries map naturally onto layered architecture.
- **Vertical Slice** (organized by feature/use case, each slice owning its own request→response path) — default for medium-sized apps where features are largely independent of each other.
- **Neither / plain structure** — for small, low-logic apps. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on Angular, the DOM, or infrastructure details. Infrastructure (HTTP clients, storage, browser APIs) implements interfaces defined by the layer/slice that needs them, not the other way around.
  
```typescript
  // Bad — domain depends on a concrete infrastructure type
  @Injectable()
  export class OrderService {
    constructor(private repository: HttpOrderRepository) {} // concrete implementation
  }

  // Good — domain depends on an abstraction it owns; infrastructure implements it
  export abstract class OrderRepository {
    abstract getById(id: string): Observable<Order | null>;
  }

  @Injectable()
  export class OrderService {
    constructor(private repository: OrderRepository) {}
  }
```

- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or the component tree, the boundary is probably wrong.

```typescript
  // Bad — the business rule can't be tested without a live data source
  @Injectable()
  export class PricingService {
    constructor(private dataSource: CustomerDataSource) {}

    calculateFinalPrice(customerId: string): Observable<number> {
      return this.dataSource
        .fetchCustomer(customerId)
        .pipe(map((customer) => (customer.isVip ? customer.basePrice * 0.9 : customer.basePrice)));
    }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  export function calculateFinalPrice(isVip: boolean, basePrice: number): number {
    return isVip ? basePrice * 0.9 : basePrice;
  }
```

- **Always depend on an abstraction, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing it; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies the same way under Clean Architecture and Vertical Slice — no carve-out for "it's simple" or "there's only one implementation today."

```typescript
  export abstract class EmailSender {
    abstract send(to: string, body: string): Promise<void>;
  }

  @Injectable()
  export class SmtpEmailSender implements EmailSender {
    async send(to: string, body: string): Promise<void> { /* ... */ }
  }

  @Injectable()
  export class OrderConfirmationHandler {
    constructor(private emailSender: EmailSender) {}
    // Testable in isolation with a fake EmailSender — no real SMTP call needed
    // to verify a confirmation gets sent after an order is placed.
  }
```

- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one feature and Vertical Slice in another within the same codebase without a deliberate, documented reason.
