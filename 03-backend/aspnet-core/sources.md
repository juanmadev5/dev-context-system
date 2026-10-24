---
tags: [aspnet-core, documentation, sources]
---

# Sources — ASP.NET Core

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs (or the relevant RFC). What counts as a source, and how to add one that isn't listed here: [sources-policy](../../00-global/sources-policy.md).

## Official domains

- For this stack, official means: `learn.microsoft.com` (ASP.NET Core, EF Core, and any other Microsoft-owned library) and each library's own docs domain listed below.

## ASP.NET Core

- Controllers / `[ApiController]` overview: <https://learn.microsoft.com/aspnet/core/web-api/>
- Dependency injection overview: <https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection>

## Authentication & authorization

- JWT Bearer authentication overview: <https://learn.microsoft.com/aspnet/core/security/authentication/jwt-authn>
- Policy-based authorization (`[Authorize]`, policies, requirements): <https://learn.microsoft.com/aspnet/core/security/authorization/policies>

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
- URL path/segment versioning for MVC controllers: <https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-url.html>

## FluentValidation

- Official documentation home: <https://docs.fluentvalidation.net/en/latest/>
- Creating your first validator: <https://docs.fluentvalidation.net/en/latest/start.html>

## xUnit

- Official documentation home: <https://xunit.net/>
- Getting Started (v3, .NET SDK command line): <https://xunit.net/docs/getting-started/v3/getting-started>

## Moq

- Official repository (devlooped/moq, the actively maintained fork): <https://github.com/devlooped/moq>
- Quickstart wiki (setup, argument matching, verification): <https://github.com/devlooped/moq/wiki/Quickstart>

## StackExchange.Redis

- Official repository and documentation: <https://github.com/StackExchange/StackExchange.Redis>
- Distributed caching in ASP.NET Core (`IDistributedCache`, Redis provider): <https://learn.microsoft.com/aspnet/core/performance/caching/distributed>

## Testcontainers for .NET

- Official documentation home: <https://dotnet.testcontainers.org/>
- PostgreSQL module: <https://dotnet.testcontainers.org/modules/postgres/>
- Redis module: <https://dotnet.testcontainers.org/modules/redis/>

## DbUp

- Official documentation home: <https://dbup.readthedocs.io/en/latest/>
- Script providers (`WithScriptsEmbeddedInAssembly`): <https://dbup.readthedocs.io/en/latest/more-info/script-providers/>

## Serilog

- Official site and documentation home: <https://serilog.net/>

## OpenTelemetry (.NET)

- .NET language docs landing page: <https://opentelemetry.io/docs/languages/dotnet/>
- Getting started guide: <https://opentelemetry.io/docs/languages/dotnet/getting-started/>

## Roslynator

- Official repository and documentation: <https://github.com/dotnet/roslynator>

## SonarAnalyzer.CSharp

- Official repository (sonar-dotnet): <https://github.com/SonarSource/sonar-dotnet>
- C# rules reference: <https://rules.sonarsource.com/csharp/>

## RFC 7807

- Problem Details for HTTP APIs — note: obsoleted by RFC 9457, worth checking if the newer RFC should be adopted instead: <https://www.rfc-editor.org/rfc/rfc7807>
