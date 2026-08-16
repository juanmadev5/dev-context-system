---
tags: [backend, aspnet-core, dotnet, csharp]
---

# ASP.NET Core

Default backend for any project that needs a dedicated backend — see [[tech-stack-map]]. A small API is not a reason to reach for [[supabase]] instead if custom business logic, non-trivial auth, or integration with S3/Keycloak is involved.

## API style — Minimal APIs vs Controllers

Same size/complexity threshold as [[architecture-principles]]'s Clean Architecture vs. Vertical Slice split — this isn't a separate decision, it's the same one applied to routing style.

- **Minimal APIs** — small/medium projects, pairs with **Vertical Slice**. Low ceremony, one endpoint file per feature fits the vertical-slice layout directly.
- **Controllers** (`[ApiController]` + attribute routing) — large apps with substantial, long-lived business logic, pairs with **Clean Architecture**. At that scale the extra ceremony pays for itself: built-in model binding/validation conventions, filters, API versioning, action-based grouping, and richer OpenAPI generation via `[ProducesResponseType]`.
- Don't mix styles within the same API — pick one per [[architecture-principles]]'s criteria and stay consistent.

## Project structure

Follow [[architecture-principles]] to pick between the two layouts below.

**Clean Architecture** (default for large apps with real, long-lived business logic):

- **Mandatory: one `.csproj` per layer, not folders inside a single project.** Physical project separation, wired together via `ProjectReference`, is what actually enforces [[architecture-principles]]'s dependency-direction rule — with folders alone, nothing stops a file under `Domain/` from importing something from `Infrastructure/` except code review catching it; with separate projects, it doesn't compile. This also means `Domain`/`Application` carry zero reference to ASP.NET Core/EF Core, so they're unit-testable with no framework/database spun up at all.

```
src/
  Domain/                # .csproj — entities, value objects, domain events, repository interfaces. No project references — zero framework deps.
  Application/            # .csproj — use cases/services, DTOs, validation, interfaces for infrastructure. References: Domain.
  Infrastructure/         # .csproj — EF Core, repository implementations, external services (S3, Keycloak, Redis). References: Application, Domain.
  Api/                    # .csproj — Controllers, DI composition root, middleware; the only executable/publishable project. References: Infrastructure, Application, Domain.
```

- This is the same shape as the well-known "Clean Architecture" .NET reference templates (Domain/Application/Infrastructure/Web) — not an unusual choice, the standard one.

**Vertical Slice** (default when a full layered split adds more ceremony than value):

```
src/
  Features/
    <Feature>/
      <Feature>Endpoint.cs   # Minimal API route mapping
      <Feature>Handler.cs    # logic for this use case
      <Feature>Request.cs / Response.cs
      <Feature>Validator.cs
  Common/                    # cross-cutting: error handling, auth policies, pagination/filtering primitives, shared infrastructure
```

## Configuration & secrets

- `appsettings.json`, `appsettings.Development.json`, and any other environment-specific variant hold **real, environment-specific values** (connection strings, credentials, API keys) — they are never committed with those values in place, same as `.env` on the frontend (see [[git-conventions]]). Add them to `.gitignore`.
- Instead, the backend project root has a **`README.md` documenting a template** for both `appsettings.json` and `appsettings.Development.json` — every key the app expects, with placeholder values (e.g. `"Default": "Host=localhost;Port=5432;Database=<db_name>;Username=<user>;Password=<password>"`) — so a new developer (or agent) can copy the template and fill in real local values without guessing the config's shape. See [[readme-conventions]] for when a per-service README like this one is warranted.
- This is the ASP.NET Core equivalent of the `.env`/`.env.example` pattern, adapted because JSON config here is layered across two files rather than one — a README is what documents the combined shape, since there isn't a single `.example` file that covers both.

## Conventions

- Endpoint routes, config keys, claim types, cache keys: constants, never magic strings — see [[coding-standards]].
- Enums serialized as strings (`.name` equivalent — `JsonStringEnumConverter`), never as raw ints, across any API boundary.
- DTOs are explicit and separate from domain entities — never expose EF Core entities directly through the API.
- Use `IOptions<T>` (or equivalent) for configuration, never `IConfiguration["Some:Key"]` scattered through the codebase.
- **Every endpoint must be explicitly and clearly identifiable, no exceptions — the exact mechanism depends on the API style:**
  - **Minimal APIs**: `.WithName("...")` on every endpoint. Naming clarity (see [[coding-standards]]) doesn't stop at variables/lambdas — an endpoint is an identifier too, and `app.MapGetTasks()` alone in `Program.cs` isn't enough: the name is what shows up in Swagger/OpenAPI as the operation ID, what typed-client generators (NSwag, openapi-typescript, etc.) key off of, what `Results.CreatedAtRoute`/`LinkGenerator` reference by, and what identifies the request in OpenTelemetry traces instead of a raw route template. Match the name to the request/query/command it dispatches (e.g. `GetTasksEndpoint` → `.WithName("GetTasks")`), not a generic or abbreviated label.
  - **Controllers**: every `[HttpGet]`/`[HttpPost]`/`[HttpPut]`/`[HttpDelete]` gets an explicit route template — never left bare relying only on the class-level `[Route("api/[controller]")]` to carry the whole path (e.g. `[HttpGet("{id}")]`, not a parameterless `[HttpGet]` on a method that clearly needs `{id}`). Reference actions via `nameof()` (`CreatedAtAction(nameof(GetById), ...)`), which is compiler-checked, rather than magic strings; add an explicit `Name = "..."` only when using `CreatedAtRoute`/`LinkGenerator` instead of `CreatedAtAction`.

## API documentation

**Every endpoint must be documented via OpenAPI — no exceptions**, same rule as [[spring-boot]].

- **`Microsoft.AspNetCore.OpenApi`** (`AddOpenApi()` / `app.MapOpenApi()`, built into the SDK since .NET 9) generates the OpenAPI document — no Swashbuckle needed just for generation.
- **Scalar** (`Scalar.AspNetCore` package, `app.MapScalarApiReference()`) serves the interactive UI at `/scalar/v1` — the default UI for new projects. Swagger UI (`Swashbuckle.AspNetCore.SwaggerUI`) is an acceptable fallback only if a project has a specific reason to keep it, not the default choice going forward.
- Every endpoint needs a summary/description, and every non-trivial DTO property needs one too — the exact mechanism depends on API style:
  - **Minimal APIs**: `.WithSummary("...")`, `.WithDescription("...")`, and `.Produces<T>(statusCode)` per possible response, chained onto every endpoint alongside the `.WithName(...)` already mandated above.
  - **Controllers**: XML doc comments (`/// <summary>`) on every action, plus `[ProducesResponseType(typeof(T), StatusCodes.Status200OK)]` for every possible response status. Set `<GenerateDocumentationFile>true</GenerateDocumentationFile>` in the `.csproj` so XML comments actually flow into the generated OpenAPI document.
- Add XML doc comments to request/response DTO properties only where the name alone doesn't say enough (units, format, constraints) — not mechanically on every property, same restraint as [[coding-standards]]'s comment guidance.
- Not optional polish: the generated spec is what a frontend/mobile consumer or an API client generator (NSwag, openapi-typescript) actually reads — an undocumented endpoint is a broken contract, not a cosmetic gap.

## API versioning

**Every endpoint is versioned via the URL path (`/api/v1/...`) — no exceptions**, same rule as [[spring-boot]]. A "v1 for now, we'll version later if we need to" mindset is how a breaking change ends up shipped straight to every existing client instead of landing behind a new version segment.

- **`Asp.Versioning.Http`** (Minimal APIs) / **`Asp.Versioning.Mvc`** (Controllers) — the maintained successor to the archived `Microsoft.AspNetCore.Mvc.Versioning`, still the standard library for this.
- **Minimal APIs**: group endpoints under a versioned `ApiVersionSet` (`app.NewApiVersionSet().HasApiVersion(new ApiVersion(1)).Build()`), route template `/api/v{version:apiVersion}/...`, `.MapToApiVersion(1)` per endpoint.
- **Controllers**: `[ApiVersion("1.0")]` on the controller class, `[Route("api/v{version:apiVersion}/[controller]")]` instead of the bare `api/[controller]` template.
- URL path segment only (`/api/v1/...`) — never a query-string (`?api-version=1`) or header-based (`X-Api-Version`) scheme as the primary mechanism; the version needs to be visible in the URL a developer pastes into a browser or shares in a bug report, not hidden in a header.
- Bump the major segment (`v1` → `v2`) only for a breaking change (removed/renamed field, changed status code, changed semantics) — additive, backward-compatible changes (new optional field, new endpoint) ship on the existing version.

## API conventions — pagination & filtering

**Every endpoint that returns a collection must be paginated. No exceptions** — a list that "currently" returns few items is still a list, and unpaginated collections are a scaling and abuse liability from day one, not something to retrofit later.

- Pagination is driven by query params: `page` (1-based, default `1`) and `pageSize` (project-defined default and hard max — e.g. default `20`, max `100` — never unbounded; a client can't request `pageSize=999999`).
- Paginated responses are always wrapped in an envelope, never a bare array, so metadata travels with the data:

  ```json
  {
    "items": [ ... ],
    "page": 1,
    "pageSize": 20,
    "totalCount": 137,
    "totalPages": 7
  }
  ```

- When the resource supports filtering, filters are query params named after the field they filter (e.g. `GET /orders?status=Active&createdAfter=2026-01-01`), applied server-side before pagination — never a single opaque "filter" blob param unless the filtering surface is generic/dynamic enough to genuinely need one.
- The pagination request/response types, the default/max page size constants, and the query-param names are defined once (in `Common`, per the Vertical Slice layout above, or in `Application`/shared kernel for Clean Architecture) and reused by every paginated endpoint — never redefined per feature, per [[coding-standards]]'s DRY rule.

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `dotnet build` — Roslyn analyzers (`Microsoft.CodeAnalysis.NetAnalyzers`) are enabled by default in SDK-style projects and run on every build; must be clean, no new warnings.
- `dotnet format --verify-no-changes` — validates style/`.editorconfig` compliance without modifying files; fails if anything would need reformatting.
- For deeper checks on large/Clean-Architecture projects, add `Roslynator` or `SonarAnalyzer.CSharp` as analyzer packages — optional, not a substitute for the two commands above.

## Pairs with

- Database: [[postgresql]] via EF Core.
- Auth: [[keycloak-auth]].
- Cache: [[redis]].
- Storage: [[aws-s3-storage]].
- Containerized in Docker for both dev and prod — see [[docker]], [[deployments]].

## Data access

- **EF Core** is the ORM/query layer only — entity configuration via **Fluent API** (`IEntityTypeConfiguration<T>` per entity), not data annotations, keeps mapping concerns out of the domain/entity classes. Table/column naming follows [[postgresql]]'s `snake_case` convention, configured explicitly rather than relying on a default naming convention that might drift.
- **Schema changes never go through `dotnet ef migrations` / EF Core Migrations.** Every SQL database backend uses **DbUp** (`dbup-postgresql`) with hand-written, versioned SQL scripts instead — this is mandatory, not a per-project choice:
  - Scripts live in `db/migration/` **in the `Infrastructure` project** (that's where the DB-access concern already lives), named `V{number}__{description}.sql` (e.g. `V001__schema.sql`, `V002__performance_indexes.sql`), zero-padded and strictly sequential. Once a script has shipped (merged, let alone deployed), it's never edited — a schema change after the fact is a new script, same as an immutable migration in any system.
  - Scripts are embedded as assembly resources (`<EmbeddedResource Include="db\migration\**\*.sql" />` in `Infrastructure.csproj`), not read from disk at runtime — `WithScriptsEmbeddedInAssembly` points at the `Infrastructure` assembly, and `Api` triggers the run at startup without needing to know where the scripts physically live.
  - Applied at startup via DbUp's `DeployChanges.To.PostgresqlDatabase(...).WithScriptsEmbeddedInAssembly(...).WithTransactionPerScript()`, gated behind a config flag (e.g. `RunMigrations`) so it can be disabled for a given environment/replica when needed.
  - Wrap the upgrade in a Postgres advisory lock (`pg_advisory_lock`/`pg_advisory_unlock` with a stable, arbitrary numeric key) around the DbUp run — this is what makes it safe for multiple replicas to start simultaneously: only one actually runs pending scripts, the rest block on the lock and find nothing pending once they get it.
  - EF Core's `DbContext`/Fluent API configuration must match the schema the SQL scripts already created — the entity mappings describe an existing schema, they never generate one. No `Migrations/` folder, no `dotnet ef database update`.

## Validation

- **FluentValidation**, one validator per request/command, decoupled from the DTO itself.

## Request flow

- **MediatR**: endpoints (Minimal API or Vertical Slice feature) dispatch a request/command to a handler via MediatR rather than calling a service directly. This keeps the endpoint thin and makes cross-cutting concerns (logging, validation, authorization) composable as pipeline behaviors instead of being repeated per endpoint.

## Error handling

- **Mandatory**: a single **`IExceptionHandler`** implementation (or one per exception category if that's cleaner), registered via `AddExceptionHandler<T>()` + `app.UseExceptionHandler()` — the equivalent centralization to [[spring-boot]]'s `@RestControllerAdvice`, one place exceptions map to HTTP responses. Never try/catch scattered per endpoint/controller action.
- Responses follow **RFC 7807 `ProblemDetails`** (`AddProblemDetails()`, `Results.Problem(...)`) — the same consistent shape as [[spring-boot]]'s convention, not an ad-hoc JSON shape per exception.
- Domain/application exceptions are specific types, never a generic `Exception` caught and stringly-typed — the handler is what maps each specific type to the right HTTP status.

## Logging & observability

- **Serilog** for structured logging (configurable sinks), **OpenTelemetry** for tracing/metrics. Every service should be able to answer "what happened to this request" without attaching a debugger.

## See also

- [[coding-standards]], [[architecture-principles]], [[readme-conventions]]
- [[postgresql]], [[keycloak-auth]], [[redis]], [[aws-s3-storage]], [[docker]]
- [[spring-boot]] (equivalent backend in the Java ecosystem)
