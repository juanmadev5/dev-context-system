---
tags: [backend, aspnet-core, dotnet, csharp]
---

# ASP.NET Core

Default backend for any project that needs a dedicated backend — see [tech-stack-map](../../00-global/tech-stack-map.md).

## API style — Controllers

Every ASP.NET Core project is exposed via **Controllers** (`[ApiController]` + attribute routing) — never Minimal APIs. This is not a per-project choice: the built-in model binding/validation conventions, filters, API versioning, action-based grouping, and richer OpenAPI generation via `[ProducesResponseType]` pay for themselves even on a small API, and pairing one consistent style with [architecture-principles](architecture-principles.md)'s always-Clean-Architecture rule keeps the stack with no secondary "which style" decision to make per project.

## Project structure

**Mandatory: one `.csproj` per layer, not folders inside a single project.** Physical project separation, wired together via `ProjectReference`, is what actually enforces [architecture-principles](architecture-principles.md)'s dependency-direction rule — with folders alone, nothing stops a file under `Domain/` from importing something from `Infrastructure/` except code review catching it; with separate projects, it doesn't compile. This also means `Domain`/`Application` carry zero reference to ASP.NET Core/EF Core, so they're unit-testable with no framework/database spun up at all.

``` text
src/
  Domain/                # .csproj — entities, value objects, domain events, repository interfaces. No project references — zero framework deps.
  Application/            # .csproj — use cases/services, DTOs, validation, interfaces for infrastructure. References: Domain.
  Infrastructure/         # .csproj — EF Core, repository implementations, external services (Redis, a payment gateway, etc.). References: Application, Domain.
  Api/                    # .csproj — Controllers, DI composition root, middleware; the only executable/publishable project. References: Infrastructure, Application, Domain.
```

This is the same shape as the well-known "Clean Architecture" .NET reference templates (Domain/Application/Infrastructure/Web) — not an unusual choice, the standard one.

## Configuration & secrets

- `appsettings.json`, `appsettings.Development.json`, and any other environment-specific variant hold **real, environment-specific values** (connection strings, credentials, API keys) — they are never committed with those values in place, same as `.env` on the frontend (see [git-conventions](../../00-global/git-conventions.md)). Add them to `.gitignore`.
- Instead, the backend project root has a **`README.md` documenting a template** for both `appsettings.json` and `appsettings.Development.json` — every key the app expects, with placeholder values (e.g. `"Default": "Host=localhost;Port=5432;Database=<db_name>;Username=<user>;Password=<password>"`) — so a new developer (or agent) can copy the template and fill in real local values without guessing the config's shape. See [readme-conventions](../../00-global/readme-conventions.md) for when a per-service README like this one is warranted.
- This is the ASP.NET Core equivalent of the `.env`/`.env.example` pattern, adapted because JSON config here is layered across two files rather than one — a README is what documents the combined shape, since there isn't a single `.example` file that covers both.

## Conventions

- Endpoint routes, config keys, claim types, cache keys: constants, never magic strings — see [engineering-principles](../../00-global/engineering-principles.md).
- **`snake_case` JSON, per [rest-api-design](../rest-api-design.md)** — set `JsonNamingPolicy.SnakeCaseLower` as the global `PropertyNamingPolicy` via `AddControllers().AddJsonOptions(...)`, instead of the framework's camelCase default. DTO properties themselves stay PascalCase (normal C# convention) — the naming policy is what converts them at the wire boundary, not a per-property `[JsonPropertyName]` attribute.
- Enums serialized as strings (`.name` equivalent — `JsonStringEnumConverter`), never as raw ints, across any API boundary.
- DTOs are explicit and separate from domain entities — never expose EF Core entities directly through the API.
- Use `IOptions<T>` (or equivalent) for configuration, never `IConfiguration["Some:Key"]` scattered through the codebase.
- **Every endpoint must be explicitly and clearly identifiable, no exceptions**: every `[HttpGet]`/`[HttpPost]`/`[HttpPut]`/`[HttpDelete]` gets an explicit route template — never left bare relying only on the class-level `[Route("api/[controller]")]` to carry the whole path (e.g. `[HttpGet("{id}")]`, not a parameterless `[HttpGet]` on a method that clearly needs `{id}`). Reference actions via `nameof()` (`CreatedAtAction(nameof(GetById), ...)`), which is compiler-checked, rather than magic strings; add an explicit `Name = "..."` only when using `CreatedAtRoute`/`LinkGenerator` instead of `CreatedAtAction`.

## Authentication & authorization

ASP.NET Core's own built-in authentication/authorization middleware (`Microsoft.AspNetCore.Authentication.JwtBearer` for token validation, policy-based `[Authorize]`) is the default — never a custom auth scheme unless there's a specific reason. **Which identity provider issues the tokens (Keycloak, Auth0, Entra ID, a project's own OIDC server, etc.) is a per-project decision, not fixed by this note** — implement whichever provider the developer specifies, pointed at that provider's own OIDC issuer/metadata endpoint.

## API documentation

Documented via OpenAPI — see [rest-api-design](../rest-api-design.md) for the mandate and the restraint rule on DTO docs; this note only adds ASP.NET Core's own library and mechanism.

- **`Microsoft.AspNetCore.OpenApi`** (`AddOpenApi()` / `app.MapOpenApi()`, built into the SDK since .NET 9) generates the OpenAPI document — no Swashbuckle needed just for generation.
- **Scalar** (`Scalar.AspNetCore` package, `app.MapScalarApiReference()`) serves the interactive UI at `/scalar/v1` — the default UI for new projects. Swagger UI (`Swashbuckle.AspNetCore.SwaggerUI`) is an acceptable fallback only if a project has a specific reason to keep it, not the default choice going forward.
- XML doc comments (`/// <summary>`) on every action, plus `[ProducesResponseType]` for every possible response status the action can actually return:

  ```csharp
  /// <summary>Creates an order placed by the authenticated user.</summary>
  [HttpPost("")]
  [ProducesResponseType(typeof(OrderDto), StatusCodes.Status201Created)]
  [ProducesResponseType(StatusCodes.Status400BadRequest)]
  public async Task<IActionResult> Create(CreateOrderRequest request) { ... }
  ```

  Set `<GenerateDocumentationFile>true</GenerateDocumentationFile>` in the `.csproj` so XML comments actually flow into the generated OpenAPI document.
- DTO property docs: XML doc comments, where [rest-api-design](../rest-api-design.md)'s restraint rule calls for one.

## API versioning

Versioned via the URL path — see [rest-api-design](../rest-api-design.md) for the mandate; this note only adds ASP.NET Core's own library and mechanism.

- **`Asp.Versioning.Mvc`** — the maintained successor to the archived `Microsoft.AspNetCore.Mvc.Versioning`, still the standard library for this.
- `[ApiVersion("1.0")]` on the controller class, `[Route("api/v{version:apiVersion}/[controller]")]` instead of the bare `api/[controller]` template.
- **`Asp.Versioning.Mvc.ApiExplorer`** is also required, with `AddApiExplorer(options => options.SubstituteApiVersionInUrl = true)` in `Program.cs`. Without it, `[ApiVersion]`/`.HasApiVersion(...)` still resolve routes correctly at runtime, but the OpenAPI document generated for Scalar keeps the literal `{version}` placeholder in every path instead of the resolved `v1`/`v2` segment — this is what actually makes the versioned routes show up correctly in the docs, not just work in the browser's address bar.

## API conventions — pagination & filtering

Paginated and filtered per [rest-api-design](../rest-api-design.md) — this note only adds where the shared types live in ASP.NET Core's own layout.

- The pagination request/response types, the default/max page size constants, and the query-param names are defined once, in `Application` (shared kernel), and reused by every paginated endpoint — never redefined per feature, per [engineering-principles](../../00-global/engineering-principles.md)'s DRY rule.

## Static analysis

Mandatory before considering any task done — see [engineering-principles](../../00-global/engineering-principles.md).

- `dotnet build` — Roslyn analyzers (`Microsoft.CodeAnalysis.NetAnalyzers`) are enabled by default in SDK-style projects and run on every build; must be clean, no new warnings.
- `dotnet format --verify-no-changes` — validates style/`.editorconfig` compliance without modifying files; fails if anything would need reformatting.
- For deeper checks on large/Clean-Architecture projects, add `Roslynator` or `SonarAnalyzer.CSharp` as analyzer packages — optional, not a substitute for the two commands above.

## Data access

- **EF Core** is the ORM/query layer only — entity configuration via **Fluent API** (`IEntityTypeConfiguration<T>` per entity), not data annotations, keeps mapping concerns out of the domain/entity classes. Table/column naming follows Postgres's `snake_case` convention, configured explicitly rather than relying on a default naming convention that might drift.
- **Schema changes never go through `dotnet ef migrations` / EF Core Migrations.** Every SQL database backend uses **DbUp** (`dbup-postgresql`) with hand-written, versioned SQL scripts instead — this is mandatory, not a per-project choice:
  - Scripts live in `db/migration/` **in the `Infrastructure` project** (that's where the DB-access concern already lives), named `V{number}__{description}.sql` (e.g. `V001__schema.sql`, `V002__performance_indexes.sql`), zero-padded and strictly sequential. Once a script has shipped (merged, let alone deployed), it's never edited — a schema change after the fact is a new script, same as an immutable migration in any system.
  - Scripts are embedded as assembly resources (`<EmbeddedResource Include="db\migration\**\*.sql" />` in `Infrastructure.csproj`), not read from disk at runtime — `WithScriptsEmbeddedInAssembly` points at the `Infrastructure` assembly, and `Api` triggers the run at startup without needing to know where the scripts physically live.
  - Applied at startup via DbUp's `DeployChanges.To.PostgresqlDatabase(...).WithScriptsEmbeddedInAssembly(...).WithTransactionPerScript()`, gated behind a config flag (e.g. `RunMigrations`) so it can be disabled for a given environment/replica when needed.
  - Wrap the upgrade in a Postgres advisory lock (`pg_advisory_lock`/`pg_advisory_unlock` with a stable, arbitrary numeric key) around the DbUp run — this is what makes it safe for multiple replicas to start simultaneously: only one actually runs pending scripts, the rest block on the lock and find nothing pending once they get it.
  - EF Core's `DbContext`/Fluent API configuration must match the schema the SQL scripts already created — the entity mappings describe an existing schema, they never generate one. No `Migrations/` folder, no `dotnet ef database update`.

## Caching

**Redis** — `Microsoft.Extensions.Caching.StackExchangeRedis` (`IDistributedCache`) for straightforward key/value caching, or `StackExchange.Redis` directly when more control is needed (pub/sub, specific data structures).

- Every cached entry has an explicit TTL appropriate to how stale the data is allowed to be — no un-expiring entries unless deliberately intended as a persistent store (which Redis isn't; that's Postgres's job).
- Redis is a cache, not a system of record — never store data there that doesn't also exist, or can't be rebuilt, in Postgres.

## Validation

- **FluentValidation**, one validator per request/command, decoupled from the DTO itself.

## Request flow

- No mediator library is used. Controllers inject `Application` services directly via constructor injection (ASP.NET Core's own DI container — no external library needed here) — the idiomatic pattern for this stack, keeping the codebase approachable.

## Error handling & logging

See [error-handling.md](error-handling.md) for the global exception-handling mechanism, structured logging, and correlation ID propagation.

## Testing

See [testing.md](testing.md) for the testing stack, structure, and mocking conventions.
