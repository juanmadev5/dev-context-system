---
tags: [flutter, testing]
---

# Testing — Flutter

Universal testing rules for every Flutter project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

## Stack

- **`flutter_test`** for unit and widget tests, **mocktail** for mocking, **`bloc_test`** for testing Bloc/Cubit state transitions instead of driving them manually through mocktail alone.

## Structure — Given-When-Then

- Tests are nested by scenario using `group`, read top-to-bottom as Given → When → Then, with the actual expectation in the innermost `test`:

```dart
group('given an order with no items', () {
  group('when processing the order', () {
    test('then throws StateError', () {
      // Given
      final order = Order(items: const []);
      final sut = OrderService(repositoryMock);

      // When
      Object? Function() act = () => sut.processOrder(order);

      // Then
      expect(act, throwsStateError);
    });
  });
});
```

- For Bloc/Cubit, `bloc_test`'s own `blocTest(...)` already expresses Given-When-Then structurally (`build`/`seed` = Given, `act` = When, `expect` = Then) — use its built-in shape rather than nesting `group`/`test` manually for state-transition tests.
- One `test`/`blocTest` per scenario — don't cram multiple Given/When combinations and several unrelated expectations into a single test.

## Mocking policy

- **Mock every injected dependency in a unit/widget test** — repositories, services, clock/time providers, anything the class under test receives via constructor/GetIt injection. A unit test never touches a real network call, platform channel, or filesystem.
- Widget tests use `pumpWidget` with mocked ViewModels (Bloc/Cubit) provided via `BlocProvider` — they verify what the widget renders for a given state, not the state logic itself (that's the Bloc/Cubit's own unit test).

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a class/use case, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
