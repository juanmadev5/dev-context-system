---
tags: [vuejs, code-review]
---

# Code Review — Vue.js

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to Vue.js and never contradicts it. The stack's static-analysis commands are in [vuejs.md](vuejs.md)'s `## Static analysis` section.

## Performance

- Unnecessary re-renders/watchers, unbounded lists without pagination/virtualization, heavy work done outside a computed/memoized boundary.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Broken access control — missing or incorrect authorization checks before a route/action is reachable (authentication alone isn't enough; guard it in the router, not just by hiding a link).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in `localStorage`/`sessionStorage` instead of a secure, httpOnly-cookie-backed flow where the backend supports one.
- XSS — raw HTML injected via `v-html` from anything user-controlled without sanitization; CSRF/CORS misconfiguration where applicable.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

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

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```typescript
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { Pending, Paid, Shipped }
  localStorage.setItem('order_status', String(order.status)); // numeric index

  // Good — stable regardless of member order
  enum OrderStatus { Pending = 'pending', Paid = 'paid', Shipped = 'shipped' }
  localStorage.setItem('order_status', order.status);
```

- Route paths, slugs, or parameter names in a language other than English (the UI language doesn't change this):

```typescript
  // Bad — route path and params follow the UI language
  export const ROUTES = { orderEdit: '/pedidos/:idPedido/editar', login: '/iniciar-sesion' };

  // Good — English, whatever language the UI text is in
  export const ROUTES = { orderEdit: '/orders/:orderId/edit', login: '/login' };
```

## Comment format

- Example of a comment that states the fix: "extract this into a composable" beats "this component is doing too much."
