---
tags: [angular, error-handling, logging]
---

# Error Handling & Logging — Angular

Universal rules for error handling and logging in every Angular project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — the HTTP interceptor's error normalization, the global error handler, and structured logging.

## Exception handling

- A central `HttpInterceptor` catches HTTP errors and normalizes them into a common `AppError` type before they reach feature services/components — features consume `AppError`, never a raw `HttpErrorResponse` deep in component code.
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable message via the normalized `AppError`.

## Global error boundary

- A custom **`ErrorHandler`** (provided app-wide via `{ provide: ErrorHandler, useClass: GlobalErrorHandler }`) catches any error Angular's own change-detection/zone machinery doesn't otherwise handle, logs it, and routes the user to a fallback state (a dedicated error UI, not a frozen/blank screen).
- This is separate from the `HttpInterceptor` above — the interceptor normalizes *expected* HTTP failures a feature can react to (validation errors, 404s), while `ErrorHandler` is the last-resort catch for anything unexpected that would otherwise surface as an unhandled exception in the console.
- No crash-reporting service (Sentry) wired up yet — the handler above logs locally and shows fallback UI; this is a deliberate current scope, not an oversight.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).
- Never log secrets, tokens, or full sensitive identifiers (passwords, card numbers) — not even at `debug` level.

## Correlation ID

- The `HttpInterceptor` attaches the current `X-Correlation-Id` to every outgoing request if one is already known for the ongoing user flow (carried over from a previous response); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side log line related to that request, and reused for an immediate retry of the same logical operation, so a failure can be traced end-to-end through the backend's own logs.
