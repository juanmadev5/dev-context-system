---
tags: [angular, code-review]
---

# Code Review — Angular

Universal rules for reviewing Angular code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command).

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null/undefined handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [[#Security checklist]] below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [[02-web/angular/architecture-principles|architecture-principles]] and the conventions in [[02-web/angular/angular|angular.md]]?
4. **Performance** — unnecessary change detection cycles, unbounded lists without pagination/virtual scrolling, missing `OnPush`/signal-based reactivity where it would matter.
5. **Tests** — is coverage present where [[02-web/angular/coding-standards|coding-standards]]'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [[02-web/angular/coding-standards|coding-standards]]), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [[02-web/angular/sources|sources]] if it actually shaped a finding.

At minimum, check for:

- Broken access control — missing or incorrect route guards or authorization checks before a component/action is reachable (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in `localStorage`/`sessionStorage` instead of a secure, httpOnly-cookie-backed mechanism.
- Missing input validation at system boundaries (see [[02-web/angular/coding-standards|coding-standards]]'s error-handling section).
- Unsafe HTML binding — `[innerHTML]` or `bypassSecurityTrust*` used on anything not already sanitized, opening the door to XSS.

Cross-reference [[04-infra/keycloak-auth|keycloak-auth]] when the project uses it.

## Anti-patterns to always flag

- God components/services doing more than one thing (violates Single Responsibility, see [[02-web/angular/coding-standards|coding-standards]]).
- Domain/business objects leaking into the presentation layer, or any other [[02-web/angular/architecture-principles|architecture-principles]] boundary violation.
  ```typescript
  // Bad — a raw HTTP response model bound straight into the template
  @Component({
    selector: 'app-customer',
    template: `<span>{{ customer()?.email }}</span>`,
  })
  export class CustomerComponent {
    customer = toSignal(this.http.get<CustomerDto>(`/api/customers/${this.id}`));
  }

  // Good — a view model shaped for the UI, decoupled from the data-layer model
  export interface CustomerViewModel {
    displayName: string;
    maskedEmail: string;
  }
  ```
- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change ([[02-web/angular/architecture-principles|architecture-principles]]). This does **not** include abstractions on injected dependencies — those are mandatory from the first implementation; never flag a DI abstraction as premature just because there's only one concrete implementation today.
- Non-descriptive callback parameter names ([[02-web/angular/coding-standards|coding-standards]]'s naming rules).
- Enums persisted or transmitted by numeric value instead of name ([[02-web/angular/coding-standards|coding-standards]]).
  ```typescript
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { Pending, Paid, Shipped }
  localStorage.setItem('order_status', String(order.status));

  // Good — stable regardless of member order
  enum OrderStatus { Pending = 'PENDING', Paid = 'PAID', Shipped = 'SHIPPED' }
  localStorage.setItem('order_status', order.status);
  ```

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "extract this into a shared component" beats "this template is too big."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm `tsc --noEmit`, `eslint`, and the test suite pass locally (see [[00-global/git-conventions|git-conventions]]).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.

## See also

- [[02-web/angular/angular|angular]]
- [[02-web/angular/coding-standards|coding-standards]]
- [[02-web/angular/architecture-principles|architecture-principles]]
- [[00-global/git-conventions|git-conventions]]
- [[02-web/angular/sources|sources]]
