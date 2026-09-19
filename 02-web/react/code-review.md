---
tags: [react, code-review]
---

# Code Review — React

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to React and never contradicts it. The stack's static-analysis commands are in [react.md](react.md)'s `## Static analysis` section.

## Performance

- Unnecessary re-renders, missing memoization on expensive computations, unpaginated large payloads.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Broken access control — missing or incorrect authorization checks per route/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in `localStorage`/`sessionStorage` instead of a secure, httpOnly-cookie-backed flow.
- XSS — unescaped user content rendered via `dangerouslySetInnerHTML`, or untrusted URLs passed straight into `href`/`src`.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

```typescript
  // Bad — a raw API/persistence shape rendered straight into the component
  function CustomerScreen({ id }: { id: string }) {
    const customer = useCustomerRecord(id); // raw API response type
    return <p>{customer.email_address}</p>;
  }

  // Good — a view model shaped for the UI, decoupled from the API's response shape
  interface CustomerViewModel {
    displayName: string;
    maskedEmail: string;
  }

  function CustomerScreen({ customer }: { customer: CustomerViewModel }) {
    return <p>{customer.maskedEmail}</p>;
  }
```

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```typescript
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { Pending, Paid, Shipped }
  localStorage.setItem("order_status", String(order.status));

  // Good — stable regardless of member order
  enum OrderStatus { Pending = "PENDING", Paid = "PAID", Shipped = "SHIPPED" }
  localStorage.setItem("order_status", order.status);
```

## Comment format

- Example of a comment that states the fix: "extract this into a custom hook" beats "this component is too big."
