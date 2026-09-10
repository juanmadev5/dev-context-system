---
tags: [spring-boot, error-handling, logging]
---

# Error Handling & Logging — Spring Boot

Universal rules for error handling and logging in every Spring Boot project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — exception types, the global handler, structured logging, and correlation IDs.

## Exception handling

- **Mandatory**: a single **`@RestControllerAdvice`** class (one per application, or per bounded context in a large Clean-Architecture project) with `@ExceptionHandler` methods per exception type — the one place exceptions get mapped to HTTP responses. Never a try/catch scattered per controller method.
- Responses follow **RFC 7807 `ProblemDetails`** (Spring's `ProblemDetail`/`ResponseEntityExceptionHandler` support) — a consistent shape (`type`, `title`, `status`, `detail`, `instance`) across every error response, not an ad-hoc JSON shape per exception.
- Domain/application exceptions are specific types (`CustomerNotFoundException`, `InsufficientBalanceException`, etc.), never a generic `RuntimeException` caught and stringly-typed — the advice class is what maps each specific type to the right HTTP status.
- Any error surfaced to a caller (API response, exception message reaching a client or log) carries a stable, predictable code (e.g. `ORDER_NOT_FOUND`, never a raw exception message) plus a human-readable message with relevant context (which order, which field) — never a generic "something went wrong".
- Never include sensitive data (passwords, tokens, secrets, full card numbers) in an error message, log entry, or exception payload — not even at debug level.

## Logging

- **SLF4J + Logback** (Spring Boot's default) for structured logging, **OpenTelemetry** (Java agent or SDK) for tracing/metrics — every service should be able to answer "what happened to this request" without attaching a debugger.
- Log levels: `ERROR` for failures that need attention (unhandled exceptions, failed external calls), `WARN` for recoverable/unexpected situations (a retried request, falling back to a default), `INFO` for business-relevant events (order created, user registered), `DEBUG` for development-only detail — never let `DEBUG`/`INFO` volume drown out `ERROR`/`WARN` in production.
- Structured logging always: parameterized log messages with named context via MDC (`MDC.put("orderId", orderId)`) rather than string-concatenated messages — fields need to stay queryable in the log sink, not buried in a formatted string.

## Correlation ID

- Every request carries an `X-Correlation-Id` header. If the client doesn't send one, the backend generates it in a `Filter` early in the chain and includes it in the response, so the caller can reuse it for related/retried requests.
- The correlation ID is pushed into SLF4J's MDC (`MDC.put("correlationId", id)`) for the lifetime of the request, so every log line emitted while handling it — including from any downstream service it calls — can be traced back to the same request. Clear it (`MDC.remove(...)`) at the end of the filter chain so it never leaks into an unrelated thread-pooled request.
