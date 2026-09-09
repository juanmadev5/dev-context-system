---
tags: [spring-boot, code-review]
---

# Code Review — Spring Boot

Universal rules for reviewing Spring Boot code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command).

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [Security checklist](#security-checklist) below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [architecture-principles](architecture-principles.md) and the conventions in [spring-boot.md](spring-boot.md)?
4. **Performance** — N+1 queries (Hibernate lazy-loading traps especially), unnecessary loops/allocations, unpaginated large payloads.
5. **Tests** — is coverage present where [coding-standards](coding-standards.md)'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [coding-standards](coding-standards.md)), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [sources](sources.md) if it actually shaped a finding.

At minimum, check for:

- Injection (SQL, command) — unparameterized queries, string-concatenated JPQL/native queries instead of `@Param`-bound queries or `Specification`s.
- Broken access control — missing or incorrect `@PreAuthorize`/method-security checks per endpoint/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or committed in `application.yml` (see [spring-boot.md](spring-boot.md)'s Configuration & secrets section).
- Missing input validation at system boundaries (see [coding-standards](coding-standards.md)'s error-handling section) — a request DTO without Bean Validation annotations.
- CORS misconfiguration on the Spring Security filter chain.

## Anti-patterns to always flag

- God classes/methods doing more than one thing (violates Single Responsibility, see [coding-standards](coding-standards.md)).
- Domain/business objects leaking into the presentation layer, or any other [architecture-principles](architecture-principles.md) boundary violation.
  
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

- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change ([architecture-principles](architecture-principles.md)). This does **not** include interfaces on injected dependencies — those are mandatory from the first implementation; never flag a DI interface as premature just because there's only one concrete implementation today.
- Non-descriptive lambda parameter names ([coding-standards](coding-standards.md)'s naming rules).
- Enums persisted or transmitted by ordinal instead of name ([coding-standards](coding-standards.md)).

```java
  // Bad — reordering or inserting a member silently changes stored meaning
  public enum OrderStatus { PENDING, PAID, SHIPPED }

  @Enumerated(EnumType.ORDINAL)
  private OrderStatus status;

  // Good — stable regardless of member order
  @Enumerated(EnumType.STRING)
  private OrderStatus status;
  ```

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "use a parameterized query here" beats "this looks unsafe."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm `mvn compile`, Checkstyle, and the test suite pass locally (see [git-conventions](../../00-global/git-conventions.md)).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.
