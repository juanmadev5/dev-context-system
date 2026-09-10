---
tags: [angular, testing]
---

# Testing — Angular

Universal testing rules for every Angular project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

## Stack

- **Jest** as the test runner (not the CLI's Jasmine/Karma default), **Angular Testing Library** for component tests (queries by role/text, never by CSS class or internal state), **`HttpTestingController`** (Angular's own, built-in) to mock HTTP calls instead of mocking `HttpClient` directly. **Playwright** for e2e when a project needs it.

## Structure — Given-When-Then

- Tests are nested by scenario using `describe`, read top-to-bottom as Given → When → Then, with the actual expectation in the innermost `it`:

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

- One `it` per scenario — don't cram multiple Given/When combinations and several unrelated assertions into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — override providers in `TestBed.configureTestingModule` for any injected service, and use `HttpTestingController` to intercept/assert on HTTP calls rather than letting a real `HttpClient` call reach the network. A unit/component test never hits a real backend.
- Component tests render through Angular Testing Library and assert on what the user actually sees (text, roles, accessible labels) — never on implementation details (a component's private fields, an injected service's internal state) that a refactor could change without changing behavior.

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a component/service, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
