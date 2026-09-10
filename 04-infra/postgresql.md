---
tags: [infra, database, postgresql]
---

# PostgreSQL

The database for **every** project, regardless of stack. Self-managed by the project's own backend, connecting to a shared local Postgres instance in development.

## Conventions

- Table/column names: `snake_case`, plural table names (e.g. `orders`, `order_items`).
- Every table has a primary key; prefer surrogate keys (`uuid` or identity) over natural keys unless there's a strong reason.
- Foreign keys are always explicit constraints, never enforced only in application code.
- Enum-like columns: use a Postgres `enum` type or a constrained/lookup table — store the semantic name, not an arbitrary integer, consistent with the project's own "enums by name" rule.
- Migrations are the only way schema changes happen — no manual/ad-hoc schema edits against a shared environment.

## Zero-downtime schema changes

- A change that breaks backward compatibility (renaming/dropping a column, changing a column's type or nullability in an incompatible way, splitting a table) is never a single migration applied directly against a table already in use — it's done via **expand-contract**, across at least two deploys:
  1. **Expand**: add the new column/table alongside the old one; the application writes to both.
  2. **Migrate reads**: once the new shape is populated and verified, the application switches to reading from it.
  3. **Contract**: only after the previous step has been live and stable, a later migration drops the old column/table.
- An additive, backward-compatible change (a new nullable column, a new table, a new index) doesn't need this — it's a normal single migration.

## Local setup

- Connects to a shared local Postgres instance rather than a project-specific one. A `keycloak` database in that instance is reserved for Keycloak — application data lives in its own database, never mixed in with `keycloak`.
- Schema migrations are versioned SQL scripts run through the project's own migration tool — the exact tool and convention live in that stack's own data-access rules.
