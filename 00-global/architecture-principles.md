---
tags: [global, architecture]
---

# Architecture Principles

## Clean Architecture vs. Vertical Slice

Both are acceptable defaults. The choice depends on project size and how much business logic it carries — not on personal preference per stack.

- **Clean Architecture** (layered: domain / application / infrastructure / presentation, dependencies pointing inward) — default for:
  - Backends with substantial business logic (e.g. an ASP.NET Core API that isn't a thin CRUD wrapper).
  - Frontends with complex business logic and long-lived state (typically Angular projects — see [[angular]]).
  - Any project expected to grow, be maintained by more than one person, or live for years.

- **Vertical Slice Architecture** (organized by feature/use case, each slice owning its own request→response path) — default for:
  - Medium-sized apps where features are largely independent of each other (typically Vue or React projects — see [[vuejs]], [[react]]).
  - Backends exposed as minimal APIs where a full layered split would add more ceremony than value, but the app still has enough logic to not be a script.

- **Neither / plain structure** — for small, low-logic projects: landing pages (Astro, HTML), simple Supabase-backed apps where Supabase itself is doing most of the heavy lifting. Don't force an architecture pattern where there's no complexity to manage.

When in doubt, pick the simpler option. Escalate to a heavier pattern only when the current structure is visibly causing friction (duplicated logic, tangled dependencies, hard-to-test business rules) — not preemptively.

## General principles (apply under either style)

- **Dependency direction**: business/domain logic never depends on frameworks, UI, or infrastructure details. Infrastructure (DB, HTTP clients, storage SDKs) implements interfaces defined by the layer/slice that needs them, not the other way around.
- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or a UI framework, the boundary is probably wrong.
- **Don't abstract prematurely**: an interface/abstraction earns its place when there are two real implementations (or a concrete, near-term need for one), not because "there might be one someday."
- **Consistency within a project beats a "better" pattern mid-stream**: don't mix Clean Architecture in one module and Vertical Slice in another within the same codebase without a deliberate, documented reason.

## See also

- [[coding-standards]]
- [[angular]], [[vuejs]], [[react]], [[aspnet-core]], [[spring-boot]]
