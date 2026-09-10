---
tags: [vuejs, code-review]
---

# Code Review — Vue.js

Universal rules for reviewing Vue code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command).

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null/undefined handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [Security checklist](#security-checklist) below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [architecture-principles](architecture-principles.md) and the conventions in [vuejs.md](vuejs.md)?
4. **Performance** — unnecessary re-renders/watchers, unbounded lists without pagination/virtualization, heavy work done outside a computed/memoized boundary.
5. **Tests** — is coverage present where [coding-standards](coding-standards.md)'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [coding-standards](coding-standards.md)), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [sources](sources.md) if it actually shaped a finding.

At minimum, check for:

- Broken access control — missing or incorrect authorization checks before a route/action is reachable (authentication alone isn't enough; guard it in the router, not just by hiding a link).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in `localStorage`/`sessionStorage` instead of a secure, httpOnly-cookie-backed flow where the backend supports one.
- Missing input validation at system boundaries (see [coding-standards](coding-standards.md)'s error-handling section).
- XSS — raw HTML injected via `v-html` from anything user-controlled without sanitization; CSRF/CORS misconfiguration where applicable.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns to always flag

- God components/composables doing more than one thing (violates Single Responsibility, see [coding-standards](coding-standards.md)).
- Domain/business objects leaking into the presentation layer, or any other [architecture-principles](architecture-principles.md) boundary violation.
  
```typescript
  // Bad — a raw API/persistence type passed straight into the component
  function useCustomer(id: string) {
    return customerApi.fetchCustomer(id); // raw API response shape
  }

  // Good — a view model shaped for the UI, decoupled from the API/persistence type
  interface CustomerViewModel {
    displayName: string;
    maskedEmail: string;
  }

  function useCustomer(id: string): Promise<CustomerViewModel> {
    return customerApi.fetchCustomer(id).then(toCustomerViewModel);
  }
```

- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change ([architecture-principles](architecture-principles.md)). This does **not** include interfaces on injected dependencies — those are mandatory from the first implementation; never flag a DI interface as premature just because there's only one concrete implementation today.
- Non-descriptive callback parameter names ([coding-standards](coding-standards.md)'s naming rules).
- Enums persisted or transmitted by numeric index instead of a stable string value ([coding-standards](coding-standards.md)).
  
```typescript
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { Pending, Paid, Shipped }
  localStorage.setItem('order_status', String(order.status)); // numeric index

  // Good — stable regardless of member order
  enum OrderStatus { Pending = 'pending', Paid = 'paid', Shipped = 'shipped' }
  localStorage.setItem('order_status', order.status);
  ```

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "extract this into a composable" beats "this component is doing too much."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm `vue-tsc`, `eslint`, and the test suite pass locally (see [git-conventions](../../00-global/git-conventions.md)).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.
