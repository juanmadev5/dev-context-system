---
tags: [aspnet-core, error-handling, logging]
---

# Error Handling & Logging — ASP.NET Core

Universal rules for error handling and logging in every ASP.NET Core project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — exception types, the global handler, structured logging, and correlation IDs.

## Exception handling

- **Mandatory**: a single **`IExceptionHandler`** implementation (or one per exception category if that's cleaner), registered via `AddExceptionHandler<T>()` + `app.UseExceptionHandler()` — one place exceptions map to HTTP responses. Never try/catch scattered per endpoint/controller action.
- Responses follow **RFC 7807 `ProblemDetails`** (`AddProblemDetails()`, `Results.Problem(...)`) — a consistent shape, not an ad-hoc JSON shape per exception.
- Domain/application exceptions are specific types, never a generic `Exception` caught and stringly-typed — the handler is what maps each specific type to the right HTTP status.
- Any error surfaced to a caller (API response, exception message reaching a client or log) carries a stable, predictable code (e.g. `ORDER_NOT_FOUND`, never a raw exception message) plus a human-readable message with relevant context (which order, which field) — never a generic "something went wrong".
- Never include sensitive data (passwords, tokens, secrets, full card numbers) in an error message, log entry, or exception payload — not even at debug level.

## Logging

- **Serilog** for structured logging (configurable sinks), **OpenTelemetry** for tracing/metrics — every service should be able to answer "what happened to this request" without attaching a debugger.
- Log levels: `Error` for failures that need attention (unhandled exceptions, failed external calls), `Warning` for recoverable/unexpected situations (a retried request, falling back to a default), `Information` for business-relevant events (order created, user registered), `Debug` for development-only detail — never let `Debug`/`Information` volume drown out `Error`/`Warning` in production.
- Structured logging always: message templates with named properties (`Log.Information("Order {OrderId} created for {CustomerId}", orderId, customerId)`), never string-interpolated messages — fields need to stay queryable in the log sink, not buried in a formatted string.

## Correlation ID

- Every request carries an `X-Correlation-Id` header. If the client doesn't send one, the backend generates it early in the middleware pipeline and includes it in the response, so the caller can reuse it for related/retried requests.
- The correlation ID is attached to Serilog's log context (`LogContext.PushProperty("CorrelationId", id)`) for the lifetime of the request, so every log line emitted while handling it — including from any downstream service it calls — can be traced back to the same request.
