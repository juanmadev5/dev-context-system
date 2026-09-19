---
tags: [angular, code-review]
---

# Code Review — Angular

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to Angular and never contradicts it. The stack's static-analysis commands are in [angular.md](angular.md)'s `## Static analysis` section.

## Performance

- Unnecessary change detection cycles, unbounded lists without pagination/virtual scrolling, missing `OnPush`/signal-based reactivity where it would matter.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Broken access control — missing or incorrect route guards or authorization checks before a component/action is reachable (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in `localStorage`/`sessionStorage` instead of a secure, httpOnly-cookie-backed mechanism.
- Unsafe HTML binding — `[innerHTML]` or `bypassSecurityTrust*` used on anything not already sanitized, opening the door to XSS.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

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

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```typescript
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { Pending, Paid, Shipped }
  localStorage.setItem('order_status', String(order.status));

  // Good — stable regardless of member order
  enum OrderStatus { Pending = 'PENDING', Paid = 'PAID', Shipped = 'SHIPPED' }
  localStorage.setItem('order_status', order.status);
```

## Comment format

- Example of a comment that states the fix: "extract this into a shared component" beats "this template is too big."
