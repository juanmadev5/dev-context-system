---
tags: [jetpack-compose, testing]
---

# Testing — Jetpack Compose

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds Jetpack Compose's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **JUnit** for logic/ViewModel tests, **Compose UI Testing** (`createComposeRule`) for Composables, **Turbine** for asserting on `Flow`/`StateFlow` emissions instead of manual `collect` boilerplate.
- **MockK** for mocking — the Kotlin-idiomatic choice over Mockito (handles `final` classes and Kotlin-specific constructs like `object`/coroutines cleanly, without the extra `mockito-inline`/`kotlin-mockito` workarounds Mockito needs).

## Structure — Given-When-Then

- Test method names read as `` given<Context>_when<Action>_then<Outcome>() `` using backtick-quoted names for readability, split into three comment blocks in the same order:

```kotlin
@Test
fun `given order has no items, when processing, then throws IllegalStateException`() {
    // Given
    val order = Order(items = emptyList())
    val sut = OrderService(repositoryMock)

    // When
    val act = { sut.processOrder(order) }

    // Then
    assertThrows<IllegalStateException> { act() }
}
```

## Mocking

- ViewModel dependencies arrive via Hilt constructor injection.
- Composable tests (`createComposeRule`) supply a fake/mocked `StateFlow` of UI state directly; the ViewModel's own logic has its separate unit test.
