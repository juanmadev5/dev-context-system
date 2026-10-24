---
tags: [backend, spring-boot, java, maven]
---

# Spring Boot

Alternative default backend to [aspnet-core](../aspnet-core/aspnet-core.md) for any project that needs a dedicated backend — see [tech-stack-map](../../00-global/tech-stack-map.md). Pick whichever of the two matches the team/project's existing language ecosystem; neither is a "smaller" or "bigger" choice than the other, they're equivalent defaults in different languages.

## Build tool

- **Maven**, always — never Gradle. Applies to every Spring Boot project, no exceptions.
- `pom.xml` is the single source of truth for dependencies/plugins; no mixing in a `build.gradle` alongside it.
- Standard lifecycle: `mvn compile`, `mvn test`, `mvn package`, `mvn verify` — scripts/docs/CI reference these, never a wrapper that hides which one is actually running.

## API style

There's no Minimal-API-style alternative in mainstream Spring — **every** endpoint is a `@RestController`. Every project uses [architecture-principles](architecture-principles.md)'s Clean Architecture module split — not a per-project choice, and it doesn't change the routing style, only the package layout underneath.

## Project structure

Every Spring Boot project follows [architecture-principles](architecture-principles.md)'s Clean Architecture layering. Controllers throughout.

- **Mandatory: one Maven module per layer, not packages inside a single module.** A multi-module project (one parent `pom.xml`, four child modules, wired via `<dependency>` between them) enforces [architecture-principles](architecture-principles.md)'s dependency-direction rule physically, not just by convention: if `domain`'s `pom.xml` declares no dependency on `infrastructure`, code in `domain` cannot import anything from it — the build fails, it isn't a code-review catch. `domain`/`application` end up with zero Spring/JPA dependency in their classpath, so they're unit-testable with nothing spun up at all.

``` text
backend/
  pom.xml                  # parent POM, declares the four modules
  domain/
    pom.xml                # no dependency on the other modules — entities, value objects, repository interfaces
  application/
    pom.xml                # <dependency>: domain — use cases/services, DTOs, validation
  infrastructure/
    pom.xml                # <dependency>: application, domain — Spring Data JPA repositories, external services (Redis, a payment gateway, etc.)
  api/
    pom.xml                # <dependency>: infrastructure, application, domain — @RestController classes, Spring config/DI, filters.
                            # The only module that's actually packaged/run (spring-boot-maven-plugin's repackage goal lives here).
```

- Not an unusual or Spring-specific stretch — multi-module Maven Clean Architecture is a well-established pattern in the Java ecosystem.

## Configuration & secrets

- `application.yml` (or `.properties`) plus environment-specific variants (`application-dev.yml`, `application-prod.yml`) hold **real, environment-specific values** — they are never committed with those values in place (see [git-conventions](../../00-global/git-conventions.md)). Add them to `.gitignore`.
- Instead, the backend project root has a **`README.md` documenting a template** for the expected config shape — every key the app expects, with placeholder values — so a new developer (or agent) can copy it and fill in real local values without guessing. See [readme-conventions](../../00-global/readme-conventions.md) for when a per-service README like this one is warranted.
- Typed config via `@ConfigurationProperties`-annotated classes bound to a config prefix, never `Environment.getProperty("some.key")` scattered through the codebase.

## Conventions

- Endpoint routes, config keys, claim types, cache keys: constants, never magic strings — see [engineering-principles](../../00-global/engineering-principles.md).
- **`snake_case` JSON, per [rest-api-design](../rest-api-design.md)** — `spring.jackson.property-naming-strategy: SNAKE_CASE` in `application.yml`, instead of Jackson's camelCase default. DTO fields themselves stay camelCase (normal Java convention) — the naming strategy is what converts them at the wire boundary, not a per-field `@JsonProperty` annotation.
- Enums serialized by name: Jackson does this by default (`Enum.name()`), so no extra configuration is needed here — just never override it to serialize by ordinal.
- DTOs (`record`s, typically) are explicit and separate from JPA `@Entity` classes — never expose an entity directly through the API.
- **Every endpoint must be explicitly and clearly identifiable, no exceptions** — every `@GetMapping`/`@PostMapping`/`@PutMapping`/`@DeleteMapping` gets its own explicit path (e.g. `@GetMapping("/{id}")`), never left bare relying only on the class-level `@RequestMapping` to carry the whole path. Naming clarity (see [engineering-principles](../../00-global/engineering-principles.md)) applies to the controller method name too — match it to what it does (`getCustomerById`, not `get2` or `handle`).
- Build response location URIs (e.g. after a `POST`) via `ServletUriComponentsBuilder` or a named route helper, not hand-concatenated strings.

## Authentication & authorization

**Spring Security** is the default security framework — the filter chain, method security (`@PreAuthorize`), and CORS configuration all go through it. **Which identity provider issues the tokens (Keycloak, Auth0, Entra ID, a project's own OIDC server, etc.) is a per-project decision, not fixed by this note** — implement whichever provider the developer specifies, typically via Spring Security's OAuth2 Resource Server support (JWT/opaque bearer token validation) pointed at that provider's own issuer.

## API documentation

Documented via OpenAPI — see [rest-api-design](../rest-api-design.md) for the mandate and the restraint rule on DTO docs; this note only adds Spring Boot's own library and mechanism.

- **springdoc-openapi** (`springdoc-openapi-starter-webmvc-ui` dependency) generates the OpenAPI 3 document and serves Swagger UI at `/swagger-ui.html` — the standard, actively-maintained library for this in Spring Boot. Springfox is legacy/unmaintained and never used on a new project.
- Every `@RestController` gets `@Tag(name = "...", description = "...")`; every endpoint method gets `@Operation(summary = "...")` plus an `@ApiResponse` per possible response status the method can actually return, including error responses mapped by the `@RestControllerAdvice`:

```java
  @Operation(summary = "Creates an order placed by the authenticated user.")
  @ApiResponse(responseCode = "201", description = "Order created",
      content = @Content(schema = @Schema(implementation = OrderDto.class)))
  @ApiResponse(responseCode = "400", description = "Invalid request body")
  @PostMapping
  public ResponseEntity<OrderDto> create(@Valid @RequestBody CreateOrderRequest request) { ... }
```

- DTO property docs: `@Schema(description = "...")`, where [rest-api-design](../rest-api-design.md)'s restraint rule calls for one.

## API versioning

Versioned via the URL path — see [rest-api-design](../rest-api-design.md) for the mandate; this note only adds Spring Boot's own mechanism.

- Spring has no equivalent to `Asp.Versioning` as an official/standard library — URI path versioning is done directly: the version prefix is baked into the class-level `@RequestMapping("/api/v1/orders")` (or a shared constant, per [engineering-principles](../../00-global/engineering-principles.md)'s magic-string rule), not added via a third-party versioning framework. The Java ecosystem doesn't converge on one versioning library the way some other ecosystems do, so this direct approach is the idiomatic default here, not a workaround.
- When a breaking change forces a `v2`, it's a **new controller class** (`OrdersV2Controller` mapped to `/api/v2/orders`), not a version parameter branching inside the same controller — keeps each version's logic independently readable and removable once the old version is retired.

## API conventions — pagination & filtering

Paginated and filtered per [rest-api-design](../rest-api-design.md) — this note only adds Spring Boot's own mechanism.

- Spring Data's own `Pageable`/`Page<T>` are the natural source of pagination (`page`, `size` query params map onto them directly via `@PageableDefault`) — but map `Page<T>` into the shared envelope shape, not Spring's native `Page` JSON structure. `page` stays 1-based on the API surface — convert to Spring's 0-based `Pageable` internally, don't leak the 0-based convention into the API.
- Filters translate into a Spring Data `Specification`/`Predicate` or a derived query method.
- The envelope DTO, default/max page size constants, and query-param names are defined once (in `application`, per the module layout above) and reused by every paginated endpoint — never redefined per feature, per [engineering-principles](../../00-global/engineering-principles.md)'s DRY rule.

## Static analysis

Mandatory before considering any task done — see [engineering-principles](../../00-global/engineering-principles.md).

- `mvn compile` — must be clean, no new compiler warnings introduced by the change.
- **Checkstyle** (`mvn checkstyle:check`, via the `maven-checkstyle-plugin`) — style/convention enforcement, the Java equivalent of `dotnet format --verify-no-changes`. Must pass clean.
- For deeper checks on large projects, add **SpotBugs** or **PMD** as additional Maven plugins — optional, not a substitute for the two commands above.

## Data access

- **Spring Data JPA** (repository interfaces extending `JpaRepository`) with Hibernate as the underlying provider.
- **JPA annotations directly on entity classes** (`@Entity`, `@Table`, `@Column`) are the idiomatic default here and are fine to use — externalized ORM mapping (`.hbm.xml`) is legacy/unusual in modern Spring, so annotating entities directly is the current, actively-maintained idiom, not a shortcut. Table/column naming still follows Postgres's `snake_case` convention (set a `PhysicalNamingStrategy`/`spring.jpa.hibernate.naming.physical-strategy` if the default doesn't already produce it, rather than annotating every single column).
- **Schema changes never go through Hibernate auto-DDL.** `spring.jpa.hibernate.ddl-auto` is always `validate` (or `none`) in every environment, including local dev — Hibernate must never create or update the schema itself. Every SQL database backend uses **Flyway** with hand-written, versioned SQL scripts instead — this is mandatory, not a per-project choice:
  - Scripts live in `db/migration/` under `src/main/resources/` of the **`api` module** (Flyway needs them on the classpath of whichever module actually boots the app), named `V{number}__{description}.sql` (e.g. `V1__schema.sql`, `V2__performance_indexes.sql`), strictly sequential. Once a script has shipped (merged, let alone deployed), it's never edited — a schema change after the fact is a new script, same as an immutable migration in any system.
  - `flyway-core` (plus `org.flywaydb:flyway-database-postgresql` for Postgres) as a dependency is all that's needed — Spring Boot auto-configures Flyway to run pending migrations at startup, gated behind `spring.flyway.enabled` so it can be disabled for a given environment/replica when needed.
  - Flyway already handles concurrent-replica-safety internally for Postgres (its own locking via the `flyway_schema_history` table) — no manual advisory-lock wiring needed.
  - Entity mappings describe an existing schema, they never generate one — the JPA annotations must match what the Flyway scripts already created.

## Caching

**Redis** — Spring Boot's caching abstraction (`@Cacheable`/`@CacheEvict`, with Redis configured as the `CacheManager` via `spring-boot-starter-data-redis`) for straightforward key/value caching, or `RedisTemplate` directly when more control is needed (pub/sub, specific data structures).

- Every cached entry has an explicit TTL appropriate to how stale the data is allowed to be — no un-expiring entries unless deliberately intended as a persistent store (which Redis isn't; that's Postgres's job).
- Redis is a cache, not a system of record — never store data there that doesn't also exist, or can't be rebuilt, in Postgres.

## Validation

- **Bean Validation** (`jakarta.validation.constraints` — `@NotBlank`, `@Email`, `@Size`, etc.) directly on request DTOs, combined with `@Valid` on the controller method parameter. This is a deliberate, stack-appropriate choice: it's the dominant, actively-maintained idiom in modern Spring Boot (built into `spring-boot-starter-validation`), even though it means validation lives on the DTO rather than in a separate validator class. Don't try to force a decoupled-validator pattern here just for the sake of a "cleaner" separation — it would fight the framework.
- For validation rules that genuinely can't be expressed as an annotation (cross-field checks, rules needing a repository lookup), use a custom `@Constraint`/`ConstraintValidator` pair, or a small dedicated method in the service layer invoked before the write — not ad-hoc `if` checks scattered through controller code.

## Request flow

- No mediator library is used. `@RestController` classes inject `@Service` beans directly via constructor injection (Spring's own DI container — no external DI library needed here) — the idiomatic Spring pattern, keeping the codebase approachable.

## Error handling & logging

See [error-handling.md](error-handling.md) for the global exception-handling mechanism, structured logging, and correlation ID propagation.

## Testing

See [testing.md](testing.md) for the testing stack, structure, and mocking conventions.
