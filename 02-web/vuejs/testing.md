---
tags: [vuejs, testing]
---

# Testing — Vue.js

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds Vue.js's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **Vitest** as the test runner, **Vue Test Utils** for component tests (queries by role/text via `@testing-library/vue` where it fits better than raw `mount`, never by CSS class or internal state), **MSW** (Mock Service Worker) to mock HTTP calls at the network level instead of mocking a composable's `fetch`/`axios` call directly. **Playwright** for e2e when a project needs it.

## Structure — Given-When-Then

- Nested with `describe`, the expectation in the innermost `it`:

```typescript
describe("given an order with no items", () => {
  describe("when submitting the order", () => {
    it("then shows a validation error", async () => {
      // Given
      const wrapper = mount(OrderForm, { props: { order: { items: [] } } });

      // When
      await wrapper.get("button[type=submit]").trigger("click");

      // Then
      expect(wrapper.text()).toContain("Order has no items");
    });
  });
});
```

## Mocking

- A composable's own dependencies (API client, other composables it calls) are mocked via `vi.mock(...)`, and any HTTP call via MSW handlers.
