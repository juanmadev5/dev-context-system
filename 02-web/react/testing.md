---
tags: [react, testing]
---

# Testing — React

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds React's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **Vitest** as the test runner, **React Testing Library** for component tests (queries by role/text, never by CSS class or internal state), **MSW** (Mock Service Worker) to mock HTTP calls at the network level instead of mocking `fetch`/`axios` directly. **Playwright** for e2e when a project needs it.

## Structure — Given-When-Then

- Nested with `describe`, the expectation in the innermost `it`:

```typescript
describe("given an order with no items", () => {
  describe("when submitting the order", () => {
    it("then shows a validation error", async () => {
      // Given
      render(<OrderForm order={{ items: [] }} />);

      // When
      await userEvent.click(screen.getByRole("button", { name: /submit/i }));

      // Then
      expect(screen.getByText(/order has no items/i)).toBeInTheDocument();
    });
  });
});
```

## Mocking

- A custom hook's own dependencies (API client, other hooks it calls) are mocked via `vi.mock(...)`, and any HTTP call via MSW handlers.
