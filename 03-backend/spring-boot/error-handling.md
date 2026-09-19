---
tags: [spring-boot, error-handling]
---

# Error Handling & Logging — Spring Boot

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: exception types, the global handler, structured logging, and correlation IDs.

## Exception handling

- **Mandatory**: a single **`@RestControllerAdvice`** class (one per application, or per bounded context in a large Clean-Architecture project) with `@ExceptionHandler` methods per exception type — the one place exceptions get mapped to HTTP responses. Never a try/catch scattered per controller method.
- Responses follow **RFC 7807 `ProblemDetails`** (Spring's `ProblemDetail`/`ResponseEntityExceptionHandler` support) — a consistent shape (`type`, `title`, `status`, `detail`, `instance`) across every error response, not an ad-hoc JSON shape per exception.

## Logging

- **SLF4J + Logback** (Spring Boot's default) for structured logging, **OpenTelemetry** (Java agent or SDK) for tracing/metrics.
- Levels: `ERROR`/`WARN`/`INFO`/`DEBUG`.
- Parameterized log messages with named context via MDC (`MDC.put("orderId", orderId)`) rather than string-concatenated messages.

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Generated in a `Filter` early in the chain when the client doesn't send one.
- Pushed into SLF4J's MDC (`MDC.put("correlationId", id)`) for the lifetime of the request. Clear it (`MDC.remove(...)`) at the end of the filter chain so it never leaks into an unrelated thread-pooled request.
