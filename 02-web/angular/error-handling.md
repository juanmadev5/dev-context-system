---
tags: [angular, error-handling]
---

# Error Handling & Logging — Angular

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: the HTTP interceptor's error normalization, the global error handler, and structured logging.

## Exception handling

- A central `HttpInterceptor` catches HTTP errors and normalizes them into a common `AppError` type before they reach feature services/components — features consume `AppError`, never a raw `HttpErrorResponse` deep in component code.

## Global error boundary

- A custom **`ErrorHandler`** (provided app-wide via `{ provide: ErrorHandler, useClass: GlobalErrorHandler }`) catches any error Angular's own change-detection/zone machinery doesn't otherwise handle, logs it, and routes the user to a fallback state (a dedicated error UI, not a frozen/blank screen).
- This is separate from the `HttpInterceptor` above — the interceptor normalizes *expected* HTTP failures a feature can react to (validation errors, 404s), while `ErrorHandler` is the last-resort catch for anything unexpected that would otherwise surface as an unhandled exception in the console.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Client side of the contract, implemented in the `HttpInterceptor`.
