---
tags: [global, code-review]
---

# Code Review

Universal rules for reviewing code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command). Applies regardless of stack; stack notes never override this.

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null/undefined handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [[#Security checklist]] below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [[architecture-principles]] and the conventions in the project's stack note?
4. **Performance** — N+1 queries, unnecessary loops/allocations, unpaginated large payloads.
5. **Tests** — is coverage present where [[coding-standards]]'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [[coding-standards]]), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [[sources-conventions]] if it actually shaped a finding.

At minimum, check for:

- Injection (SQL, NoSQL, command, LDAP) — unparameterized queries, string-concatenated commands.
- Broken access control — missing or incorrect authorization checks per endpoint/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded or logged.
- Missing input validation at system boundaries (see [[coding-standards]]'s error-handling section).
- CSRF/CORS misconfiguration, where applicable to the stack.

Cross-reference [[keycloak-auth]] or [[supabase-auth]] when the project uses either.

## Anti-patterns to always flag

- God classes/functions doing more than one thing (violates Single Responsibility, see [[coding-standards]]).
- Domain/business objects leaking into the presentation layer, or any other [[architecture-principles]] boundary violation.
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
- Premature abstraction — an interface or split introduced with no real second implementation ([[architecture-principles]]).
- Non-descriptive lambda/callback parameter names ([[coding-standards]]'s naming rules).
- Enums persisted or transmitted by ordinal instead of name ([[coding-standards]]).
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
- Confirm linting, type-checking, and the test suite pass locally (see [[git-conventions]]).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.

## See also

- [[coding-standards]]
- [[architecture-principles]]
- [[git-conventions]]
- [[sources-conventions]]
