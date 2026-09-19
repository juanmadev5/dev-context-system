---
tags: [global, testing]
---

# Testing Principles

How a test is written once it's warranted — the same for every stack. *When* a test is mandatory is decided in [engineering-principles](engineering-principles.md)'s `## Testing` section. Each stack's `testing.md` only adds its own libraries, naming format, code example, and mocking mechanics, and never contradicts this note.

## Structure — Given-When-Then

- Every test reads as Given → When → Then, and its body is split into three commented blocks in that order (`// Given`, `// When`, `// Then`), replacing the generic Arrange/Act/Assert labels.
- Where the framework lets blocks nest (`describe`/`group`), nest by scenario read top-to-bottom as Given → When → Then, with the actual expectation in the innermost block. Where it doesn't, the test name carries the same structure (`given<Context>_when<Action>_then<Outcome>`) — the exact format is in the stack's `testing.md`.
- One test per scenario — don't cram multiple Given/When combinations and several unrelated assertions into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — repositories, services, external clients, clock/time providers, everything the class under test receives via injection. A unit test never touches a real database, network call, platform channel, or filesystem.
- Integration tests are the ones that exercise real infrastructure — they trade the isolation of mocks for confidence that the actual persistence mapping and schema behave correctly against the real thing. Don't mix the two styles in the same test class.
- UI/component tests assert on what the user actually sees (rendered text, roles, accessible labels) for a given state — never on implementation details (private state, internal fields, props) that a refactor could change without changing behavior. The state logic itself is covered by its own unit test.

## What to test

- Prefer testing through the public API of a class/use case/component, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per the stack's `sources.md`.
