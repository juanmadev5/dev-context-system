---
tags: [jetpack-compose, code-review]
---

# Code Review — Jetpack Compose

Stack-specific additions to [code-review](../../00-global/code-review.md) — read that first; this note only adds what's specific to Jetpack Compose and never contradicts it. The stack's static-analysis commands are in [jetpack-compose.md](jetpack-compose.md)'s `## Static analysis` section.

## Performance

- Unnecessary recompositions, unbounded lists without paging/lazy loading, heavy work off the coroutine dispatcher meant for it.

## Security

On top of the minimum in [code-review](../../00-global/code-review.md)'s security checklist, check for:

- Broken access control — missing or incorrect authorization checks before a screen/action is reachable (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in plain `SharedPreferences` instead of encrypted `DataStore`/`EncryptedSharedPreferences`.
- Insecure network calls — no certificate pinning/TLS validation bypass, no secrets embedded in request URLs.

See also [security-practices](../../04-infra/security-practices.md) for the standing infra-level conventions (secrets, dependency hygiene) this checklist assumes are already in place.

## Anti-patterns

- Persistence/API objects leaking into the presentation or API layer:

```kotlin
  // Bad — a data-layer entity rendered straight in the Composable
  @Composable
  fun CustomerScreen(customerId: String, dataSource: CustomerDataSource) {
      val customer = dataSource.fetchCustomer(customerId) // raw DTO
      Text(customer.email)
  }

  // Good — a UI state shaped for the screen, decoupled from the data-layer model
  data class CustomerUiState(val displayName: String, val maskedEmail: String)

  @Composable
  fun CustomerScreen(uiState: CustomerUiState) {
      Text(uiState.maskedEmail)
  }
```

- Enums persisted or transmitted by ordinal/index/numeric value instead of name:

```kotlin
  // Bad — reordering or inserting a member silently changes stored meaning
  enum class OrderStatus { PENDING, PAID, SHIPPED }
  sharedPreferences.edit { putInt("order_status", order.status.ordinal) }

  // Good — stable regardless of member order
  sharedPreferences.edit { putString("order_status", order.status.name) }
```

## Comment format

- Example of a comment that states the fix: "extract this into a named Composable" beats "this composable is too big."
