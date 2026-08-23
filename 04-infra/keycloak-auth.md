---
tags: [infra, auth, keycloak]
---

# Keycloak Auth

Auth provider for every project — see [[00-global/tech-stack-map|tech-stack-map]].

## Local setup

- Provided by [[04-infra/local-infrastructure|local-infrastructure]]: Keycloak `start-dev` mode, admin console on `http://localhost:8080/` (port configurable via `KEYCLOAK_PORT`), state persisted in its own `keycloak` Postgres database plus its own data volume.
- `start-dev` is for local development only (HTTP, relaxed config) — never used as-is as a production template.

## Conventions

- The backend (ASP.NET Core's JWT bearer middleware, or Spring Security's `spring-boot-starter-oauth2-resource-server`) validates tokens against the Keycloak realm's OIDC issuer (`http://<host>:<port>/realms/<realm>` locally) — standard OpenID Connect / JWT bearer validation, no custom auth scheme unless there's a specific reason.
- Roles/permissions come from Keycloak realm/client roles mapped into JWT claims — authorization checks in the backend read from claims, not from a parallel roles table unless there's a real need to store app-specific permissions Keycloak doesn't model well.
- Realm, client IDs, and claim type names: constants, never magic strings, per the project's stack's own no-magic-values rule.
- Production Keycloak configuration (real TLS, hardened realm settings, persistent admin credentials) is a deliberate separate setup from local `start-dev` — never copy local `.env` defaults (`admin`/`admin`) into a real environment.

## See also

- [[04-infra/local-infrastructure|local-infrastructure]], [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]
