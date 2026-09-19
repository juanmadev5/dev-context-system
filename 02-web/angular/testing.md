---
tags: [angular, testing]
---

# Testing — Angular

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds Angular's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **Jest** as the test runner (not the CLI's Jasmine/Karma default), **Angular Testing Library** for component tests (queries by role/text, never by CSS class or internal state), **`HttpTestingController`** (Angular's own, built-in) to mock HTTP calls instead of mocking `HttpClient` directly. **Playwright** for e2e when a project needs it.

## Structure — Given-When-Then

- Nested with `describe`, the expectation in the innermost `it`:

```typescript
describe("given an order with no items", () => {
  describe("when submitting the order", () => {
    it("then shows a validation error", async () => {
      // Given
      await render(OrderFormComponent, { inputs: { order: { items: [] } } });

      // When
      await userEvent.click(screen.getByRole("button", { name: /submit/i }));

      // Then
      expect(screen.getByText(/order has no items/i)).toBeInTheDocument();
    });
  });
});
```

## Mocking

- Override providers in `TestBed.configureTestingModule` for any injected service, and use `HttpTestingController` to intercept/assert on HTTP calls rather than letting a real `HttpClient` call reach the network.
