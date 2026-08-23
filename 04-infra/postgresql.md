---
tags: [infra, database, postgresql]
---

# PostgreSQL

The database for **every** project, regardless of stack — see [[00-global/tech-stack-map|tech-stack-map]]. Self-managed via [[03-backend/aspnet-core/aspnet-core|aspnet-core]] or [[03-backend/spring-boot/spring-boot|spring-boot]], connecting to [[04-infra/local-infrastructure|local-infrastructure]] locally.

## Conventions

- Table/column names: `snake_case`, plural table names (e.g. `orders`, `order_items`).
- Every table has a primary key; prefer surrogate keys (`uuid` or identity) over natural keys unless there's a strong reason.
- Foreign keys are always explicit constraints, never enforced only in application code.
- Enum-like columns: use a Postgres `enum` type or a constrained/lookup table — store the semantic name, not an arbitrary integer, consistent with the project's stack's own "enums by name" rule.
- Migrations are the only way schema changes happen — no manual/ad-hoc schema edits against a shared environment.

## Local setup

- Connects to the shared Postgres instance in [[04-infra/local-infrastructure|local-infrastructure]]. The `keycloak` database in that instance is reserved for Keycloak — application data lives in its own database, never mixed in with `keycloak`. Schema migrations are versioned SQL scripts run through **DbUp** (ASP.NET Core) or **Flyway** (Spring Boot) — see that stack's own Data access section for the exact convention.

## See also

- [[04-infra/local-infrastructure|local-infrastructure]], [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]
