---
tags: [infra, database, postgresql]
---

# PostgreSQL

The database for **every** project, regardless of stack — see [[tech-stack-map]]. Either self-managed (via [[aspnet-core]] or [[spring-boot]], connecting to [[local-infrastructure]] locally) or Supabase-managed (see [[supabase]]).

## Conventions

- Table/column names: `snake_case`, plural table names (e.g. `orders`, `order_items`).
- Every table has a primary key; prefer surrogate keys (`uuid` or identity) over natural keys unless there's a strong reason.
- Foreign keys are always explicit constraints, never enforced only in application code.
- Enum-like columns: use a Postgres `enum` type or a constrained/lookup table — store the semantic name, not an arbitrary integer, consistent with [[coding-standards]]'s "enums by name" rule.
- Migrations are the only way schema changes happen — no manual/ad-hoc schema edits against a shared environment.

## Local vs. Supabase

- **Local/self-managed** (paired with [[aspnet-core]] or [[spring-boot]]): connects to the shared Postgres instance in [[local-infrastructure]]. The `keycloak` database in that instance is reserved for Keycloak — application data lives in its own database, never mixed in with `keycloak`. Schema migrations are versioned SQL scripts run through **DbUp** (ASP.NET Core) or **Flyway** (Spring Boot) — see that stack's own Data access section for the exact convention.
- **Supabase-managed** (paired with [[supabase]]): schema lives in the Supabase project; RLS policies (see [[supabase]]) are the authorization layer on top of the same conventions above.

## See also

- [[local-infrastructure]], [[aspnet-core]], [[spring-boot]], [[supabase]]
