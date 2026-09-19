---
tags: [aspnet-core, error-handling]
---

# Error Handling & Logging — ASP.NET Core

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: exception types, the global handler, structured logging, and correlation IDs.

## Exception handling

- **Mandatory**: a single **`IExceptionHandler`** implementation (or one per exception category if that's cleaner), registered via `AddExceptionHandler<T>()` + `app.UseExceptionHandler()` — one place exceptions map to HTTP responses. Never try/catch scattered per endpoint/controller action.
- Responses follow **RFC 7807 `ProblemDetails`** (`AddProblemDetails()`, `Results.Problem(...)`) — a consistent shape, not an ad-hoc JSON shape per exception.

## Logging

- **Serilog** for structured logging (configurable sinks), **OpenTelemetry** for tracing/metrics.
- Levels: `Error`/`Warning`/`Information`/`Debug`.
- Message templates with named properties (`Log.Information("Order {OrderId} created for {CustomerId}", orderId, customerId)`), never string-interpolated messages.

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Generated in middleware early in the pipeline when the client doesn't send one.
- Attached to Serilog's log context (`LogContext.PushProperty("CorrelationId", id)`) for the lifetime of the request.
