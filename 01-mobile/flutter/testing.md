---
tags: [flutter, testing]
---

# Testing — Flutter

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds Flutter's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **`flutter_test`** for unit and widget tests, **mocktail** for mocking, **`bloc_test`** for testing Bloc/Cubit state transitions instead of driving them manually through mocktail alone.

## Structure — Given-When-Then

- Nested with `group`, the expectation in the innermost `test`:

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

## Mocking

- Dependencies arrive via constructor/GetIt injection.
- Widget tests use `pumpWidget` with mocked ViewModels (Bloc/Cubit) provided via `BlocProvider`; the Bloc/Cubit's state logic has its own unit test.
