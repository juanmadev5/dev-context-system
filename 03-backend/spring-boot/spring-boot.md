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
alquilapy-backend/
  pom.xml                  # parent POM, declares the four modules
  domain/
    pom.xml                # no dependency on the other modules — entities, value objects, repository interfaces
  application/
    pom.xml                # <dependency>: domain — use cases/services, DTOs, validation
  infrastructure/
    pom.xml                # <dependency>: application, domain — Spring Data JPA repositories, external services (S3, Keycloak, Redis)
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

- Endpoint routes, config keys, claim types, cache keys: constants, never magic strings — see [coding-standards](coding-standards.md).
- Enums serialized by name: Jackson does this by default (`Enum.name()`), so no extra configuration is needed here — just never override it to serialize by ordinal.
- DTOs (`record`s, typically) are explicit and separate from JPA `@Entity` classes — never expose an entity directly through the API.
- **Every endpoint must be explicitly and clearly identifiable, no exceptions** — every `@GetMapping`/`@PostMapping`/`@PutMapping`/`@DeleteMapping` gets its own explicit path (e.g. `@GetMapping("/{id}")`), never left bare relying only on the class-level `@RequestMapping` to carry the whole path. Naming clarity (see [coding-standards](coding-standards.md)) applies to the controller method name too — match it to what it does (`getCustomerById`, not `get2` or `handle`).
- Build response location URIs (e.g. after a `POST`) via `ServletUriComponentsBuilder` or a named route helper, not hand-concatenated strings.

## API documentation

**Every endpoint must be documented via OpenAPI/Swagger — no exceptions.**

- **springdoc-openapi** (`springdoc-openapi-starter-webmvc-ui` dependency) generates the OpenAPI 3 document and serves Swagger UI at `/swagger-ui.html` — the standard, actively-maintained library for this in Spring Boot. Springfox is legacy/unmaintained and never used on a new project.
- Every `@RestController` gets `@Tag(name = "...", description = "...")`; every endpoint method gets `@Operation(summary = "...")` plus an `@ApiResponse` per possible response status the method can actually return — success and error alike, including error responses mapped by the `@RestControllerAdvice`:

```java
  @Operation(summary = "Creates a property owned by the authenticated user.")
  @ApiResponse(responseCode = "201", description = "Property created",
      content = @Content(schema = @Schema(implementation = PropertyDto.class)))
  @ApiResponse(responseCode = "400", description = "Invalid request body")
  @PostMapping
  public ResponseEntity<PropertyDto> create(@Valid @RequestBody CreatePropertyRequest request) { ... }
```

- Add `@Schema(description = "...")` to request/response DTO properties only where the name alone doesn't say enough (units, format, constraints) — not mechanically on every field, same restraint as [coding-standards](coding-standards.md)'s comment guidance.
- Not optional polish: the generated spec is what a frontend/mobile consumer or an API client generator actually reads — an undocumented endpoint is a broken contract, not a cosmetic gap.

## API versioning

**Every endpoint is versioned via the URL path (`/api/v1/...`) — no exceptions.** A "v1 for now, we'll version later if we need to" mindset is how a breaking change ends up shipped straight to every existing client instead of landing behind a new version segment.

- Spring has no equivalent to `Asp.Versioning` as an official/standard library — URI path versioning is done directly: the version prefix is baked into the class-level `@RequestMapping("/api/v1/orders")` (or a shared constant, per [coding-standards](coding-standards.md)'s magic-string rule), not added via a third-party versioning framework. The Java ecosystem doesn't converge on one versioning library the way some other ecosystems do, so this direct approach is the idiomatic default here, not a workaround.
- When a breaking change forces a `v2`, it's a **new controller class** (`OrdersV2Controller` mapped to `/api/v2/orders`), not a version parameter branching inside the same controller — keeps each version's logic independently readable and removable once the old version is retired.
- URL path segment only — never a query-string or header-based scheme as the primary mechanism; the version needs to be visible in the URL a developer pastes into a browser or shares in a bug report, not hidden in a header.
- Bump the major segment (`v1` → `v2`) only for a breaking change (removed/renamed field, changed status code, changed semantics) — additive, backward-compatible changes (new optional field, new endpoint) ship on the existing version.

## API conventions — pagination & filtering

**Every endpoint that returns a collection must be paginated. No exceptions** — a list that "currently" returns few items is still a list, and unpaginated collections are a scaling and abuse liability from day one, not something to retrofit later.

- Spring Data's own `Pageable`/`Page<T>` are the natural source of pagination (`page`, `size` query params map onto them directly via `@PageableDefault`) — but **map `Page<T>` into the same envelope shape used across every stack in this repository**, not Spring's native `Page` JSON structure, so the API contract stays identical regardless of which backend a given project uses:

```json
  {
    "items": [ ... ],
    "page": 1,
    "pageSize": 20,
    "totalCount": 137,
    "totalPages": 7
  }
```

- Query param names: `page` (1-based — convert to Spring's 0-based `Pageable` internally, don't leak the 0-based convention into the API surface), `pageSize` (project-defined default and hard max, e.g. default `20`, max `100`, never unbounded).
- Filters are query params named after the field they filter (e.g. `GET /orders?status=Active&createdAfter=2026-01-01`), translated into a Spring Data `Specification`/`Predicate` or a derived query method — applied server-side before pagination, never a single opaque "filter" blob param unless the filtering surface is genuinely generic/dynamic.
- The envelope DTO, default/max page size constants, and query-param names are defined once (in `application`, per the module layout above) and reused by every paginated endpoint — never redefined per feature, per [coding-standards](coding-standards.md)'s DRY rule.

## Static analysis

Mandatory before considering any task done — see [coding-standards](coding-standards.md).

- `mvn compile` — must be clean, no new compiler warnings introduced by the change.
- **Checkstyle** (`mvn checkstyle:check`, via the `maven-checkstyle-plugin`) — style/convention enforcement, the Java equivalent of `dotnet format --verify-no-changes`. Must pass clean.
- For deeper checks on large projects, add **SpotBugs** or **PMD** as additional Maven plugins — optional, not a substitute for the two commands above.

## Pairs with

- Database: [postgresql](../../04-infra/postgresql.md) via Spring Data JPA / Hibernate.
- Auth: [keycloak-auth](../../04-infra/keycloak-auth.md) — an especially natural pairing (Keycloak integrates directly with Spring Security via `spring-boot-starter-oauth2-resource-server`).
- Cache: [redis](../../04-infra/redis.md) via Spring Data Redis / `spring-boot-starter-cache`.
- Storage: [aws-s3-storage](../../04-infra/aws-s3-storage.md) via the AWS SDK for Java v2.
- Containerized in Docker for both dev and prod — see [docker](../../04-infra/docker.md).

## Data access

- **Spring Data JPA** (repository interfaces extending `JpaRepository`) with Hibernate as the underlying provider.
- **JPA annotations directly on entity classes** (`@Entity`, `@Table`, `@Column`) are the idiomatic default here and are fine to use — externalized ORM mapping (`.hbm.xml`) is legacy/unusual in modern Spring, so annotating entities directly is the current, actively-maintained idiom, not a shortcut. Table/column naming still follows [postgresql](../../04-infra/postgresql.md)'s `snake_case` convention (set a `PhysicalNamingStrategy`/`spring.jpa.hibernate.naming.physical-strategy` if the default doesn't already produce it, rather than annotating every single column).
- **Schema changes never go through Hibernate auto-DDL.** `spring.jpa.hibernate.ddl-auto` is always `validate` (or `none`) in every environment, including local dev — Hibernate must never create or update the schema itself. Every SQL database backend uses **Flyway** with hand-written, versioned SQL scripts instead — this is mandatory, not a per-project choice:
  - Scripts live in `db/migration/` under `src/main/resources/` of the **`api` module** (Flyway needs them on the classpath of whichever module actually boots the app), named `V{number}__{description}.sql` (e.g. `V1__schema.sql`, `V2__performance_indexes.sql`), strictly sequential. Once a script has shipped (merged, let alone deployed), it's never edited — a schema change after the fact is a new script, same as an immutable migration in any system.
  - `flyway-core` (plus `org.flywaydb:flyway-database-postgresql` for Postgres) as a dependency is all that's needed — Spring Boot auto-configures Flyway to run pending migrations at startup, gated behind `spring.flyway.enabled` so it can be disabled for a given environment/replica when needed.
  - Flyway already handles concurrent-replica-safety internally for Postgres (its own locking via the `flyway_schema_history` table) — no manual advisory-lock wiring needed.
  - Entity mappings describe an existing schema, they never generate one — the JPA annotations must match what the Flyway scripts already created.

## Validation

- **Bean Validation** (`jakarta.validation.constraints` — `@NotBlank`, `@Email`, `@Size`, etc.) directly on request DTOs, combined with `@Valid` on the controller method parameter. This is a deliberate, stack-appropriate choice: it's the dominant, actively-maintained idiom in modern Spring Boot (built into `spring-boot-starter-validation`), even though it means validation lives on the DTO rather than in a separate validator class. Don't try to force a decoupled-validator pattern here just for the sake of a "cleaner" separation — it would fight the framework.
- For validation rules that genuinely can't be expressed as an annotation (cross-field checks, rules needing a repository lookup), use a custom `@Constraint`/`ConstraintValidator` pair, or a small dedicated method in the service layer invoked before the write — not ad-hoc `if` checks scattered through controller code.

## Request flow

- No MediatR-equivalent is mandated. `@RestController` classes inject `@Service` beans directly via constructor injection (Spring's own DI container — no external DI library needed here) — this is the idiomatic Spring pattern and keeps the codebase approachable; introducing a mediator library would add ceremony without a corresponding ecosystem convention behind it.

## Error handling

- **Mandatory**: a single **`@RestControllerAdvice`** class (one per application, or per bounded context in a large Clean-Architecture project) with `@ExceptionHandler` methods per exception type — this is the one place exceptions get mapped to HTTP responses. Never a try/catch scattered per controller method.
- Responses follow **RFC 7807 `ProblemDetails`** (Spring's `ProblemDetail`/`ResponseEntityExceptionHandler` support) — a consistent shape (`type`, `title`, `status`, `detail`, `instance`) across every error response, not an ad-hoc JSON shape per exception.
- Domain/application exceptions are specific types (`CustomerNotFoundException`, `InsufficientBalanceException`, etc.), never a generic `RuntimeException` caught and stringly-typed — the advice class is what maps each specific type to the right HTTP status.

## Logging & observability

- **SLF4J + Logback** (Spring Boot's default) for structured logging, **OpenTelemetry** (Java agent or SDK) for tracing/metrics — every service should be able to answer "what happened to this request" without attaching a debugger.

## Testing

- **JUnit 5 + Mockito** is the idiomatic default when tests are warranted — see [coding-standards](coding-standards.md)'s Testing section for when they're actually mandatory (backend testing stack, if any, is chosen per-project). `spring-boot-starter-test` bundles both plus AssertJ.
