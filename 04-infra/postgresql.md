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

## Local setup

- Connects to a shared local Postgres instance rather than a project-specific one. A `keycloak` database in that instance is reserved for Keycloak — application data lives in its own database, never mixed in with `keycloak`.
- Schema migrations are versioned SQL scripts run through the project's own migration tool — the exact tool and convention live in that stack's own data-access rules.
