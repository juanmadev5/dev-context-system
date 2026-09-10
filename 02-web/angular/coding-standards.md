---
tags: [angular, coding-standards]
---

# Coding Standards — Angular

Universal rules that apply to every Angular project. [angular.md](angular.md) only adds on top of this — it never contradicts it.

## Language

- All code, identifiers, comments, commit messages, and documentation are written in **English**.
- The only exception is text that is user-facing in the UI (labels, messages, validation text), which follows the project's target language(s).
- If the backend returns semantic codes/keys instead of literal text (e.g. `ERROR_INVALID_CREDENTIALS`) and the frontend maps them to localized strings, the codes themselves are still English constants — only the mapped UI text is localized.

## Naming

- Every identifier must be understandable **at first glance**, with no need to trace back through the code to figure out what it represents. This includes variables, parameters, and — just as strictly — **callback parameters** in `.filter`/`.map`/`.reduce`-style chains and RxJS operators (`.pipe(map(...), filter(...))`).
- Never name a callback parameter after a generic placeholder letter (`e`, `x`, `el`, `arr`) when a descriptive name is one keystroke away.
- Bad (meaning only recoverable by re-reading the surrounding types):
  
```typescript
  const errorsByField = validationErrors
    .filter((e) => e.severity === 'error')
    .reduce<Record<string, string[]>>((acc, e) => {
      (acc[e.field] ??= []).push(e.message);
      return acc;
    }, {});
```

- Good (each name says what the item is, no guessing required):
  
```typescript
  const errorsByField = validationErrors
    .filter((validationError) => validationError.severity === 'error')
    .reduce<Record<string, string[]>>((accumulated, validationError) => {
      (accumulated[validationError.field] ??= []).push(validationError.message);
      return accumulated;
    }, {});
```

- The one broadly accepted exception is a raw numeric index in a tight loop (`for (let i = 0; ...)`). Everything else — including nested callback parameters shadowing an outer one — gets a real, descriptive name.

## No magic values

- Never inline literal strings or numbers that carry meaning (route paths, HTTP header names, storage keys, config values, thresholds, etc.).
- Always extract them to named constants, or to a TypeScript `enum` (or a `const`-object union) when the value represents a closed set of options.
- When persisting or transmitting an enum (local storage, JSON, query params), always use its **name**, never its numeric value. Numeric values silently break when a member is added, removed, or reordered; names are stable and self-documenting across boundaries.

## DRY

- Duplication is a defect, not a style preference. If the same logic (not just similar-looking code) appears more than once, extract it — a function, a shared service, a base class, a utility module, whatever fits.
- DRY applies to logic and business rules, not to superficial structural similarity. Do not force an abstraction over code that merely looks alike but represents different concerns — that creates false coupling. See [architecture-principles](architecture-principles.md) for how this interacts with premature abstraction.

## SOLID

Applied together with DRY, not instead of it — see [architecture-principles](architecture-principles.md) for how these interact with layer/slice boundaries.

- **Single Responsibility**: a class/component/function has one reason to change. If describing what something does requires "and", it's a candidate to split.
- **Open/Closed**: extend behavior by adding new code (a new implementation of an interface, a new case), not by modifying working code to special-case a new scenario — especially across module boundaries.
- **Liskov Substitution**: a subtype/implementation must be usable anywhere its base type/interface is expected, without the caller needing to know which concrete type it got. If a caller has to type-check or special-case a specific implementation, the abstraction is wrong.
- **Interface Segregation**: don't force a consumer to depend on methods it doesn't use. Prefer several small, focused interfaces/abstract classes over one large one.
- **Dependency Inversion**: high-level/business logic depends on abstractions, not on concrete infrastructure. Same rule as [architecture-principles](architecture-principles.md)'s "dependency direction" — SOLID and the architecture layering reinforce each other, they're not separate concerns.

Apply these pragmatically: they're a guide for keeping code changeable, not a checklist to satisfy for its own sake. This doesn't apply to injected dependencies — those always get an abstraction, from the first implementation, per [architecture-principles](architecture-principles.md)'s dependency rule. It's about internal structure: don't split a class into multiple pieces or add extra indirection with no real boundary or reason to change — that's premature abstraction, which [architecture-principles](architecture-principles.md) already warns against.

## Architecture

- Default to **Clean Architecture** or **Vertical Slice** depending on project size/complexity. See [architecture-principles](architecture-principles.md) for the decision criteria and layout.
- **Never create a folder that ends up empty.** The `core/`, `shared/`, `features/<feature>/` tree shown in [angular.md](angular.md) is the *shape* a project converges toward, not a scaffold to stamp out up front — create a folder only at the moment it actually gets its first file.
- Corollary: don't pre-create the full folder tree for a new feature "so it's ready" — add each folder as the corresponding file is written. An empty folder in the repo is either dead weight (most VCS don't even track it) or, worse, a placeholder someone has to remember to clean up.

## Comments

- Default to no comments. Code should be self-explanatory through naming. This stays the default for trivial code — getters, direct mappings, anything a well-named signature already explains.
- Write a comment when it captures a **non-obvious why**: a hidden constraint, a workaround for a specific bug/API quirk, a business rule that isn't derivable from the code itself.
- **Exception to "never comment on what"**: add a short JSDoc (`/** */`) comment above a function when it's genuinely complex or abstract enough that a competent reader can't infer its purpose/approach from the name + signature + a read of the body alone. This is not an invitation to comment everything:
  - Recursive logic or non-trivial algorithms (backtracking, graph traversal, DP).
  - Multi-step chains of higher-order functions or RxJS operators (e.g. a `.pipe(switchMap(...), map(...), catchError(...))` pipeline with several transformations stacked).
  - Generic/abstract code (complex generics, decorator-based metaprogramming, Strategy/Visitor-style services where intent isn't clear without seeing how it's used).
  - Rule of thumb: if describing what the function does in one sentence would require walking through more than 2-3 chained steps, or the function's name doesn't communicate *how* it achieves its result, it qualifies.
  - Format: JSDoc `/** */` above the declaration — 1-3 lines, summarizing purpose and approach, not a line-by-line narration. Comments inside the function body are still off the table for this case.

## Error handling

- Validate and handle errors at system boundaries (user input, external API responses, browser APIs). Don't add defensive checks for states that are impossible given internal invariants already enforced by TypeScript's type system.
- Fail loudly in development; degrade gracefully (with proper logging) in production paths that face end users.
- See [error-handling](error-handling.md) for the HTTP interceptor's error normalization, the global error handler, and structured logging.

## Definition of done

A task is never "done" just because it behaves correctly or compiles. Code can look fine and still be silently broken, scoped wrong, or undocumented — before considering any task/feature finished, every applicable item below must be checked, not just a feeling that it's "probably fine":

- **Static analysis** — passes clean (no new errors or warnings introduced by the change) — see [angular.md](angular.md)'s `## Static analysis` section. Never skip this assuming "it looks fine" or because the change was small.
- **Tests** — pass locally; if the change touches logic covered by this note's [Testing](#testing) criteria, new tests were written for it (see [testing](testing.md) for how).
- **Self-review** — the full diff was read start to finish before calling the task done, per [code-review](code-review.md)'s self-review section.
- **Docs** — the project's README was updated if the change affects it ([readme-conventions](../../00-global/readme-conventions.md)); `docs/SOURCES.md` was updated if a source not already covered by [sources](sources.md) was consulted.
- **Scope check** — the change matches exactly what was asked, with no unrelated edits left in (see [Scope discipline](#scope-discipline) below).
- **No residue** — no leftover debug code, commented-out blocks, or unowned TODOs.

## Testing

- Tests are **not a blanket requirement for every project or every piece of logic.** Writing tests for a trivial CRUD screen or low-stakes glue code is its own form of over-engineering — see Scope discipline below.
- Tests **are mandatory** for business logic that's genuinely delicate and error-prone: money/billing calculations, complex state transitions, permission/authorization logic, concurrency-sensitive code, or anything where a silent bug would corrupt data or cause a real incident rather than just a cosmetic glitch.
- The judgment call: "if this breaks silently, how bad is it?" — if the honest answer involves someone getting paid wrong, a user seeing another user's data, or a state machine landing in an invalid state, it needs tests. If the worst case is "a list renders in the wrong order," it probably doesn't.
- See [testing](testing.md) for the testing stack, structure, and mocking conventions used once tests are warranted.

## Scope discipline

- Implement what the task requires. Don't add speculative flexibility, extra config options, or abstractions for hypothetical future needs.
- No half-finished implementations: either a feature is complete for its intended scope, or it isn't started.

## Ask vs. assume

- **Ask, and don't proceed until answered**, when: the decision is business/domain-specific and not inferable from the existing code or this repository (e.g. what should happen when a field is null in a specific business flow); multiple reasonable interpretations exist with materially different outcomes (breaking vs. additive change, the shape of a data model). Check the project's `CLAUDE.md` first — don't ask what's already documented there.
- **Decide and proceed**, when: it's a routine implementation detail with one obviously-correct answer given the codebase's existing patterns (naming a variable, extracting a duplicate); it's already resolved by a repository note or the project's `CLAUDE.md`.
- **Never guess an API, method, or parameter** that hasn't been verified against real code or official documentation ([sources](sources.md)) — that's always a case to check, never to fabricate.
- If proceeding on a judgment call rather than asking, state the assumption explicitly (in the PR description or a note to the developer) instead of deciding silently. Silent, unstated assumptions are exactly what produce a "reasonable but wrong" decision that only surfaces at review.
