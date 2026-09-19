---
tags: [spring-boot, code-review]
---

# Code Review — Spring Boot

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to Spring Boot and never contradicts it. The stack's static-analysis commands are in [spring-boot.md](spring-boot.md)'s `## Static analysis` section.

## Performance

- N+1 queries (Hibernate lazy-loading traps especially), unnecessary loops/allocations, unpaginated large payloads.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Injection (SQL, command) — unparameterized queries, string-concatenated JPQL/native queries instead of `@Param`-bound queries or `Specification`s.
- Broken access control — missing or incorrect `@PreAuthorize`/method-security checks per endpoint/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or committed in `application.yml` (see [spring-boot.md](spring-boot.md)'s Configuration & secrets section).
- Missing input validation at system boundaries — a request DTO without Bean Validation annotations.
- CORS misconfiguration on the Spring Security filter chain.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, rate limiting, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

```java
  // Bad — the JPA entity returned straight from the controller
  @GetMapping("/{id}")
  public Customer get(@PathVariable UUID id) {
      return customerRepository.findById(id).orElseThrow();
  }

  // Good — a DTO shaped for the API contract, decoupled from the persistence model
  @GetMapping("/{id}")
  public CustomerResponse get(@PathVariable UUID id) {
      Customer customer = customerRepository.findById(id).orElseThrow();
      return new CustomerResponse(customer.getId(), customer.getName(), customer.getEmail());
  }
```

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```java
  // Bad — reordering or inserting a member silently changes stored meaning
  public enum OrderStatus { PENDING, PAID, SHIPPED }

  @Enumerated(EnumType.ORDINAL)
  private OrderStatus status;

  // Good — stable regardless of member order
  @Enumerated(EnumType.STRING)
  private OrderStatus status;
```
