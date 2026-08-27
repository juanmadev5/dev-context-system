---
tags: [aspnet-core, documentation, sources]
---

# Sources — ASP.NET Core

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs (or the relevant RFC).

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `learn.microsoft.com` (ASP.NET Core, EF Core, and any other Microsoft-owned library) and each library's own docs domain listed below.

## ASP.NET Core

- Minimal APIs overview and quick reference: <https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis>
- Controllers / `[ApiController]` overview: <https://learn.microsoft.com/aspnet/core/web-api/>
- Dependency injection overview: <https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection>

## EF Core

- Fluent API model configuration, `IEntityTypeConfiguration<TEntity>` grouping pattern: <https://learn.microsoft.com/ef/core/modeling/>
- `IEntityTypeConfiguration<TEntity>` interface reference: <https://learn.microsoft.com/dotnet/api/microsoft.entityframeworkcore.ientitytypeconfiguration-1>

## Microsoft.AspNetCore.OpenApi

- OpenAPI support overview in ASP.NET Core: <https://learn.microsoft.com/aspnet/core/fundamentals/openapi/overview>
- Generate OpenAPI documents (`AddOpenApi`, `MapOpenApi`): <https://learn.microsoft.com/aspnet/core/fundamentals/openapi/aspnetcore-openapi>

## Scalar

- ASP.NET Core integration (`Scalar.AspNetCore`, `MapScalarApiReference`): <https://guides.scalar.com/scalar/scalar-api-references/net-integration>

## Asp.Versioning

- Getting started with `Asp.Versioning.Http`: <https://dotnet.github.io/aspnet-api-versioning/getting-started.html>
- URL path/segment versioning for Minimal APIs and MVC controllers: <https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-url.html>

## FluentValidation

- Official documentation home: <https://docs.fluentvalidation.net/en/latest/>
- Creating your first validator: <https://docs.fluentvalidation.net/en/latest/start.html>

## xUnit

- Official documentation home: <https://xunit.net/>
- Getting Started (v3, .NET SDK command line): <https://xunit.net/docs/getting-started/v3/getting-started>

## Moq

- Official repository (devlooped/moq, the actively maintained fork): <https://github.com/devlooped/moq>
- Quickstart wiki (setup, argument matching, verification): <https://github.com/devlooped/moq/wiki/Quickstart>

## DbUp

- Official documentation home: <https://dbup.readthedocs.io/en/latest/>
- Script providers (`WithScriptsEmbeddedInAssembly`): <https://dbup.readthedocs.io/en/latest/more-info/script-providers/>

## Serilog

- Official site and documentation home: <https://serilog.net/>

## OpenTelemetry (.NET)

- .NET language docs landing page: <https://opentelemetry.io/docs/languages/dotnet/>
- Getting started guide: <https://opentelemetry.io/docs/languages/dotnet/getting-started/>

## RFC 7807

- Problem Details for HTTP APIs — note: obsoleted by RFC 9457, worth checking if the newer RFC should be adopted instead: <https://www.rfc-editor.org/rfc/rfc7807>

## If a source isn't listed here

This index covers the libraries/topics this repository's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [coding-standards](coding-standards.md)'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

```markdown
  ## SomeNewPackage

  - What the page covers: https://...
```
