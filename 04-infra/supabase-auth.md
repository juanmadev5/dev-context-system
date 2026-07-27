---
tags: [infra, auth, supabase]
---

# Supabase Auth

Auth provider for Supabase-only projects — see [[tech-stack-map]], [[supabase]]. Use [[keycloak-auth]] instead once the project has a dedicated [[aspnet-core]] or [[spring-boot]] backend.

## Conventions

- Authorization is enforced via Postgres Row Level Security policies keyed off `auth.uid()` (or custom claims), not just checked client-side — see [[supabase]].
- Custom claims/roles needed beyond Supabase's defaults go through Auth Hooks or a dedicated table joined against `auth.users`, kept consistent with [[coding-standards]] (named constants/enums for role values, never magic strings).
- Session handling uses Supabase's client SDK defaults (refresh tokens, storage) unless a specific project constraint requires overriding them.

## See also

- [[supabase]], [[postgresql]]
- [[keycloak-auth]]
