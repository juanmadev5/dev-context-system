---
tags: [jetpack-compose, testing]
---

# Testing — Jetpack Compose

Universal testing rules for every Jetpack Compose project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

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

- One `@Test` per scenario — don't cram multiple Given/When combinations and several unrelated assertions into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — repositories, use cases, clock/time providers, anything the ViewModel receives via Hilt constructor injection. A ViewModel unit test never touches a real network call or database.
- Composable tests (`createComposeRule`) supply a fake/mocked `StateFlow` of UI state directly — they verify what the Composable renders for a given state, not the ViewModel's own logic (that's the ViewModel's separate unit test).

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a class/use case, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
