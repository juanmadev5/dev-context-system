---
tags: [spring-boot, coding-standards]
---

# Coding Standards — Spring Boot

Universal rules that apply to every Spring Boot project. [spring-boot.md](spring-boot.md) only adds on top of this — it never contradicts it.

## Language

- All code, identifiers, comments, commit messages, and documentation are written in **English**.
- The only exception is text that is user-facing in the UI (labels, messages, validation text), which follows the project's target language(s).
- If the backend returns semantic codes/keys instead of literal text (e.g. `ERROR_INVALID_CREDENTIALS`) and the frontend maps them to localized strings, the codes themselves are still English constants — only the mapped UI text is localized.

## Naming

- Every identifier must be understandable **at first glance**, with no need to trace back through the code to figure out what it represents. This includes variables, parameters, and — just as strictly — **lambda parameters** in stream chains (`.map`/`.filter`/`.collect`/`Collectors.groupingBy`-style pipelines).
- Never name a lambda parameter after a generic placeholder letter (`e`, `x`, `el`, `arr`, `i` for anything but a raw loop index) when a descriptive name is one keystroke away.
- Bad (meaning only recoverable by re-reading the surrounding types):
  
```java
  var errors = validationErrors.stream()
      .collect(Collectors.groupingBy(
          e -> e.getField(),
          Collectors.mapping(e -> e.getMessage(), Collectors.toList())));
```

- Good (each name says what the item is, no guessing required):

```java
  var errorsByField = validationErrors.stream()
      .collect(Collectors.groupingBy(
          validationError -> validationError.getField(),
          Collectors.mapping(validationError -> validationError.getMessage(), Collectors.toList())));
```

- The one broadly accepted exception is a raw numeric index in a tight loop (`for (int i = 0; ...)`). Everything else — including nested lambda parameters shadowing an outer one — gets a real, descriptive name.
- Avoid generic, content-free names (`data`, `result`, `item`, `obj`, `temp`) for any variable or parameter whenever a name that describes what the value actually represents is available — `pendingOrder`, not `order`, once there's more than one order-shaped value in scope. A generic name forces the reader to trace the code back to figure out what it holds; a descriptive one says it up front.

## Control flow

- Prefer **guard clauses** over nested conditionals: check failure/exit conditions first and return or throw immediately, so the method body isn't wrapped in an ever-deepening `if`. The main logic should read top-to-bottom with no more than one level of nesting for the happy path.

```java
  // Bad — main logic buried inside nested conditions
  public Order processOrder(Order order) {
      if (order != null) {
          if (!order.getItems().isEmpty()) {
              if (order.getStatus() == OrderStatus.PENDING) {
                  return repository.save(order);
              }
          }
      }
      throw new IllegalStateException("Cannot process order");
  }

  // Good — guard clauses exit early, main logic is flat and visible
  public Order processOrder(Order order) {
      if (order == null) throw new IllegalArgumentException("order must not be null");
      if (order.getItems().isEmpty()) throw new IllegalStateException("Order has no items");
      if (order.getStatus() != OrderStatus.PENDING) throw new IllegalStateException("Order is not pending");

      return repository.save(order);
  }
```

## No magic values

- Never inline literal strings or numbers that carry meaning (endpoint routes, config keys, claim types, cache keys, thresholds, etc.).
- Always extract them to named constants, or to an `enum` when the value represents a closed set of options.
- When persisting or transmitting an enum (DB, JSON, query params), always use its **name**, never its ordinal. Ordinals silently break when a member is added, removed, or reordered; names are stable and self-documenting across service boundaries.

## DRY

- Duplication is a defect, not a style preference. If the same logic (not just similar-looking code) appears more than once, extract it — a method, a shared component, a base class, a utility class, whatever fits.
- DRY applies to logic and business rules, not to superficial structural similarity. Do not force an abstraction over code that merely looks alike but represents different concerns — that creates false coupling. See [architecture-principles](architecture-principles.md) for how this interacts with premature abstraction.

## SOLID

Applied together with DRY, not instead of it — see [architecture-principles](architecture-principles.md) for how these interact with layer/slice boundaries.

- **Single Responsibility**: a class/method has one reason to change. If describing what something does requires "and", it's a candidate to split.
- **Open/Closed**: extend behavior by adding new code (a new implementation of an interface, a new case), not by modifying working code to special-case a new scenario — especially across module boundaries.
- **Liskov Substitution**: a subtype/implementation must be usable anywhere its base type/interface is expected, without the caller needing to know which concrete type it got. If a caller has to type-check or special-case a specific implementation, the abstraction is wrong.
- **Interface Segregation**: don't force a consumer to depend on methods it doesn't use. Prefer several small, focused interfaces over one large one.
- **Dependency Inversion**: high-level/business logic depends on abstractions, not on concrete infrastructure. Same rule as [architecture-principles](architecture-principles.md)'s "dependency direction" — SOLID and the architecture layering reinforce each other, they're not separate concerns.

Apply these pragmatically: they're a guide for keeping code changeable, not a checklist to satisfy for its own sake. This doesn't apply to injected dependencies — those always get an interface, from the first implementation, per [architecture-principles](architecture-principles.md)'s dependency rule. It's about internal structure: don't split a class into multiple pieces or add extra indirection with no real boundary or reason to change — that's premature abstraction, which [architecture-principles](architecture-principles.md) already warns against.

## Architecture

- Default to **Clean Architecture** for every project — see [architecture-principles](architecture-principles.md) for the layout. Not a per-project choice.
- **Never create a package/module that ends up empty.** The `domain`/`application`/`infrastructure`/`api` module split shown in [spring-boot.md](spring-boot.md) is the *shape* a project converges toward, not a scaffold to stamp out up front — create a package or module only at the moment it actually gets its first class.
- Corollary: don't pre-create the full module/package tree for a new feature "so it's ready" — add each package as the corresponding class is written. An empty package in the repo is either dead weight or, worse, a placeholder someone has to remember to clean up.

## Comments

- Default to no comments. Code should be self-explanatory through naming. This stays the default for trivial code — getters, direct mappings, anything a well-named signature already explains.
- Write a comment when it captures a **non-obvious why**: a hidden constraint, a workaround for a specific bug/API quirk, a business rule that isn't derivable from the code itself.
- **Exception to "never comment on what"**: add a short Javadoc comment above a method when it's genuinely complex or abstract enough that a competent reader can't infer its purpose/approach from the name + signature + a read of the body alone. This is not an invitation to comment everything:
  - Recursive logic or non-trivial algorithms (backtracking, graph traversal, DP).
  - Multi-step chains of higher-order functions (e.g. a `.stream().map().collect()`-style pipeline with several transformations stacked).
  - Generic/abstract code (complex generics, reflection, Strategy/Visitor-style patterns where intent isn't clear without seeing how it's used).
  - Rule of thumb: if describing what the method does in one sentence would require walking through more than 2-3 chained steps, or the method's name doesn't communicate *how* it achieves its result, it qualifies.
  - Format: `/** ... */` Javadoc above the declaration — 1-3 lines, summarizing purpose and approach, not a line-by-line narration. Comments inside the method body are still off the table for this case.

## Error handling

- Validate and handle errors at system boundaries (user input, external API responses, I/O). Don't add defensive checks for states that are impossible given internal invariants already enforced by the type system or Bean Validation.
- Fail loudly in development; degrade gracefully (with proper logging) in production paths that face end users.
- See [error-handling](error-handling.md) for the global exception-handling mechanism, structured logging, and correlation ID propagation.

## Definition of done

A task is never "done" just because it behaves correctly or compiles. Code can look fine and still be silently broken, scoped wrong, or undocumented — before considering any task/feature finished, every applicable item below must be checked, not just a feeling that it's "probably fine":

- **Static analysis** — passes clean (no new errors or warnings introduced by the change) — see [spring-boot.md](spring-boot.md)'s `## Static analysis` section. Never skip this assuming "it looks fine" or because the change was small.
- **Tests** — pass locally; if the change touches logic covered by this note's [Testing](#testing) criteria, new tests were written for it (see [testing](testing.md) for how).
- **Self-review** — the full diff was read start to finish before calling the task done, per [code-review](code-review.md)'s self-review section.
- **Docs** — the project's README was updated if the change affects it ([readme-conventions](../../00-global/readme-conventions.md)); `docs/SOURCES.md` was updated if a source not already covered by [sources](sources.md) was consulted.
- **Scope check** — the change matches exactly what was asked, with no unrelated edits left in (see [Scope discipline](#scope-discipline) below).
- **No residue** — no leftover debug code, commented-out blocks, or unowned TODOs.

## Testing

- Tests are **not a blanket requirement for every project or every piece of logic.** Writing tests for a trivial CRUD endpoint or low-stakes glue code is its own form of over-engineering — see Scope discipline below.
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
