---
tags: [jetpack-compose, code-review]
---

# Code Review — Jetpack Compose

Universal rules for reviewing Jetpack Compose code — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command).

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [Security checklist](#security-checklist) below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in [architecture-principles](architecture-principles.md) and the conventions in [jetpack-compose.md](jetpack-compose.md)?
4. **Performance** — unnecessary recompositions, unbounded lists without paging/lazy loading, heavy work off the coroutine dispatcher meant for it.
5. **Tests** — is coverage present where [coding-standards](coding-standards.md)'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [coding-standards](coding-standards.md)), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` per [sources](sources.md) if it actually shaped a finding.

At minimum, check for:

- Broken access control — missing or incorrect authorization checks before a screen/action is reachable (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded, logged, or stored in plain `SharedPreferences` instead of encrypted `DataStore`/`EncryptedSharedPreferences`.
- Missing input validation at system boundaries (see [coding-standards](coding-standards.md)'s error-handling section).
- Insecure network calls — no certificate pinning/TLS validation bypass, no secrets embedded in request URLs.

Cross-reference [keycloak-auth](../../04-infra/keycloak-auth.md) when the project uses it.

## Anti-patterns to always flag

- God Composables/functions doing more than one thing (violates Single Responsibility, see [coding-standards](coding-standards.md)).
- Domain/business objects leaking into the presentation layer, or any other [architecture-principles](architecture-principles.md) boundary violation.

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

- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change ([architecture-principles](architecture-principles.md)). This does **not** include interfaces on injected dependencies — those are mandatory from the first implementation; never flag a DI interface as premature just because there's only one concrete implementation today.
- Non-descriptive lambda parameter names ([coding-standards](coding-standards.md)'s naming rules).
- Enums persisted or transmitted by `.ordinal` instead of `.name` ([coding-standards](coding-standards.md)).

```kotlin
  // Bad — reordering or inserting a member silently changes stored meaning
  enum class OrderStatus { PENDING, PAID, SHIPPED }
  sharedPreferences.edit { putInt("order_status", order.status.ordinal) }

  // Good — stable regardless of member order
  sharedPreferences.edit { putString("order_status", order.status.name) }
```

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "extract this into a named Composable" beats "this composable is too big."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm `./gradlew lint`, `./gradlew detekt`, and the test suite pass locally (see [git-conventions](../../00-global/git-conventions.md)).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.
