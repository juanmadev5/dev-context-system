---
tags: [aspnet-core, code-review]
---

# Code Review — ASP.NET Core

Universal rules for reviewing ASP.NET Core code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command).

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null/undefined handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [Security checklist](#security-checklist) below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [architecture-principles](architecture-principles.md) and the conventions in [aspnet-core.md](aspnet-core.md)?
4. **Performance** — N+1 queries, unnecessary loops/allocations, unpaginated large payloads.
5. **Tests** — is coverage present where [coding-standards](coding-standards.md)'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [coding-standards](coding-standards.md)), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [sources](sources.md) if it actually shaped a finding.

At minimum, check for:

- Injection (SQL, NoSQL, command, LDAP) — unparameterized queries, string-concatenated commands.
- Broken access control — missing or incorrect authorization checks per endpoint/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded or logged.
- Missing input validation at system boundaries (see [coding-standards](coding-standards.md)'s error-handling section).
- CSRF/CORS misconfiguration, where applicable.

Cross-reference [keycloak-auth](../../04-infra/keycloak-auth.md) when the project uses it.

## Anti-patterns to always flag

- God classes/functions doing more than one thing (violates Single Responsibility, see [coding-standards](coding-standards.md)).
- Domain/business objects leaking into the presentation layer, or any other [architecture-principles](architecture-principles.md) boundary violation.

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

- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change ([architecture-principles](architecture-principles.md)). This does **not** include interfaces on injected dependencies — those are mandatory from the first implementation; never flag a DI interface as premature just because there's only one concrete implementation today.
- Non-descriptive lambda/callback parameter names ([coding-standards](coding-standards.md)'s naming rules).
- Enums persisted or transmitted by ordinal instead of name ([coding-standards](coding-standards.md)).

```csharp
  // Bad — reordering or inserting a member silently changes stored meaning
  public enum OrderStatus { Pending, Paid, Shipped }
  context.SaveJson(new { status = (int)order.Status });

  // Good — stable regardless of member order
  context.SaveJson(new { status = order.Status.ToString() });
  ```

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "use parameterized query here" beats "this looks unsafe."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm `dotnet build`/`dotnet format --verify-no-changes` and the test suite pass locally (see [git-conventions](../../00-global/git-conventions.md)).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.
