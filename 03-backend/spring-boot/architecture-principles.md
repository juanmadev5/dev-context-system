---
tags: [spring-boot, architecture]
---

# Architecture Principles — Spring Boot

## Clean Architecture

Every Spring Boot project uses **Clean Architecture** — layered: domain / application / infrastructure / presentation, dependencies pointing inward — regardless of project size. This is not a per-project choice.

## General principles

- **Dependency direction**: business/domain logic never depends on Spring, JPA, or infrastructure details. Infrastructure (Spring Data repositories, HTTP clients, external service SDKs) implements interfaces defined by the layer/slice that needs them, not the other way around.

```java
  // Bad — domain depends on a concrete infrastructure type
  @Service
  public class OrderService {
      private final JpaOrderRepository repository; // concrete Spring Data JPA class

      public OrderService(JpaOrderRepository repository) {
          this.repository = repository;
      }
  }

  // Good — domain depends on an abstraction it owns; infrastructure implements it
  public interface OrderRepository {
      Optional<Order> findById(UUID id);
  }

  @Service
  public class OrderService {
      private final OrderRepository repository;

      public OrderService(OrderRepository repository) {
          this.repository = repository;
      }
  }
```

- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or the Spring context, the boundary is probably wrong.

```java
  // Bad — the business rule can't be tested without a live data source
  @Service
  public class PricingService {
      public BigDecimal calculateFinalPrice(UUID customerId) {
          Customer customer = customerRepository.findById(customerId).orElseThrow();
          return customer.isVip() ? customer.getBasePrice().multiply(BigDecimal.valueOf(0.9)) : customer.getBasePrice();
      }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  public final class PricingRules {
      public static BigDecimal calculateFinalPrice(boolean isVip, BigDecimal basePrice) {
          return isVip ? basePrice.multiply(BigDecimal.valueOf(0.9)) : basePrice;
      }
  }
```

- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies at every layer boundary — no carve-out for "it's simple" or "there's only one implementation today."

```java
  public interface EmailSender {
      void send(String to, String body);
  }

  @Service
  public class SmtpEmailSender implements EmailSender {
      public void send(String to, String body) { /* ... */ }
  }

  @Service
  public class OrderConfirmationHandler {
      private final EmailSender emailSender;
      public OrderConfirmationHandler(EmailSender emailSender) {
          this.emailSender = emailSender;
      }
      // Testable in isolation with a fake EmailSender — no real SMTP call needed
      // to verify a confirmation gets sent after an order is placed.
  }
```

- **Consistency within a project beats a "better" pattern mid-stream**: every module respects the same domain/application/infrastructure boundaries — don't let one module skip a layer "because it's simple" while the rest keep the full split.
