---
tags: [flutter, code-review]
---

# Code Review — Flutter

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to Flutter and never contradicts it. The stack's static-analysis commands are in [flutter.md](flutter.md)'s `## Static analysis` section.

## Performance

- Unnecessary rebuilds, unbounded lists without pagination/lazy loading, heavy work on the UI thread.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Broken access control — missing or incorrect authorization checks before a screen/action is reachable (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in plain `SharedPreferences` instead of secure storage.
- Insecure network calls — no certificate pinning/TLS validation bypass, no secrets embedded in request URLs.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

```dart
  // Bad — a data-layer model returned straight to the widget tree
  class CustomerScreen extends StatelessWidget {
    Widget build(BuildContext context) {
      final customer = customerDataSource.fetchCustomer(id); // raw DTO
      return Text(customer.email);
    }
  }

  // Good — a view model shaped for the UI, decoupled from the data-layer model
  class CustomerViewModel {
    final String displayName;
    final String maskedEmail;
    CustomerViewModel({required this.displayName, required this.maskedEmail});
  }
```

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```dart
  // Bad — reordering or inserting a member silently changes stored meaning
  enum OrderStatus { pending, paid, shipped }
  prefs.setInt('order_status', order.status.index);

  // Good — stable regardless of member order
  prefs.setString('order_status', order.status.name);
```

## Comment format

- Example of a comment that states the fix: "extract this into a named widget" beats "this build method is too big."
