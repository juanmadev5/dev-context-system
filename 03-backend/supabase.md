---
tags: [backend, supabase, postgresql]
---

# Supabase

Default when a project is simple enough that a dedicated backend would be overkill — see [[tech-stack-map]]. Once the project needs real custom business logic, non-trivial auth flows, or infrastructure beyond what Supabase provides, move to [[aspnet-core]] or [[spring-boot]] instead of stretching Supabase (Edge Functions, complex RLS) to cover it.

## What it replaces

- Database: still [[postgresql]] (Supabase-managed).
- Auth: [[supabase-auth]] instead of [[keycloak-auth]].
- Storage: [[supabase-storage]] instead of [[aws-s3-storage]].
- No separate backend service, no [[docker]] backend container — the client talks to Supabase directly.

## Conventions

- Row Level Security (RLS) policies are the primary authorization boundary — every table exposed to the client must have explicit RLS policies, never rely on the client to "just not query" restricted data.
- Business logic that must run server-side (not safely doable via RLS/client) goes in Supabase Edge Functions or Postgres functions — not duplicated/trusted client-side.
- Keep policy and function names, and any custom Postgres enums, following [[coding-standards]] (no magic values, explicit naming).
- Treat this as a genuine architectural choice, not a shortcut: if the project outgrows what RLS + Edge Functions can cleanly express, that's the signal to introduce [[aspnet-core]] or [[spring-boot]] rather than piling on complexity here.

## Client & types

- Generate TypeScript types directly from the live schema (`supabase gen types typescript`) and commit them or regenerate them as part of the build — never hand-write types that mirror the DB schema, they will drift.

## Edge Functions

- One function per use case (vertical-slice style), not large multi-route functions with an internal router. Shared concerns (input validation helpers, the Supabase client setup, common types) live in a `_shared` module imported by each function, per [[coding-standards]]'s DRY rule — never copy-pasted across functions.

## See also

- [[coding-standards]]
- [[postgresql]], [[supabase-auth]], [[supabase-storage]]
- [[vuejs]] (typical frontend pairing for small/medium apps)
