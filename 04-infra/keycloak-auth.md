---
tags: [infra, auth, keycloak]
---

# Keycloak Auth

Auth provider when the project has a dedicated [[aspnet-core]] or [[spring-boot]] backend — see [[tech-stack-map]]. Use [[supabase-auth]] instead for Supabase-only projects.

## Local setup

- Provided by [[local-infrastructure]]: Keycloak `start-dev` mode, admin console on `http://localhost:8080/` (port configurable via `KEYCLOAK_PORT`), state persisted in its own `keycloak` Postgres database plus its own data volume.
- `start-dev` is for local development only (HTTP, relaxed config) — never used as-is as a production template.

## Conventions

- The backend (ASP.NET Core's JWT bearer middleware, or Spring Security's `spring-boot-starter-oauth2-resource-server`) validates tokens against the Keycloak realm's OIDC issuer (`http://<host>:<port>/realms/<realm>` locally) — standard OpenID Connect / JWT bearer validation, no custom auth scheme unless there's a specific reason.
- Roles/permissions come from Keycloak realm/client roles mapped into JWT claims — authorization checks in the backend read from claims, not from a parallel roles table unless there's a real need to store app-specific permissions Keycloak doesn't model well.
- Realm, client IDs, and claim type names: constants, never magic strings, per [[coding-standards]].
- Production Keycloak configuration (real TLS, hardened realm settings, persistent admin credentials) is a deliberate separate setup from local `start-dev` — never copy local `.env` defaults (`admin`/`admin`) into a real environment.

## See also

- [[local-infrastructure]], [[aspnet-core]], [[spring-boot]]
- [[supabase-auth]]
