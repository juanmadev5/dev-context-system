---
tags: [vuejs, testing]
---

# Testing — Vue.js

Universal testing rules for every Vue project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

## Stack

- **Vitest** as the test runner, **Vue Test Utils** for component tests (queries by role/text via `@testing-library/vue` where it fits better than raw `mount`, never by CSS class or internal state), **MSW** (Mock Service Worker) to mock HTTP calls at the network level instead of mocking a composable's `fetch`/`axios` call directly. **Playwright** for e2e when a project needs it.

## Structure — Given-When-Then

- Tests are nested by scenario using `describe`, read top-to-bottom as Given → When → Then, with the actual expectation in the innermost `it`:

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

- One `it` per scenario — don't cram multiple Given/When combinations and several unrelated assertions into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — a composable's own dependencies (API client, other composables it calls) via `vi.mock(...)`, and any HTTP call via MSW handlers. A unit/component test never hits a real backend.
- Component tests assert on what the user actually sees (rendered text, roles, accessible labels) — never on implementation details (a composable's internal ref, a component's private state) that a refactor could change without changing behavior.

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a component/composable, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
