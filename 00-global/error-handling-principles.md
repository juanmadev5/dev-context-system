---
tags: [global, error-handling, logging]
---

# Error Handling & Logging Principles

The policy that's the same for every stack. *Where* boundary validation happens is in [engineering-principles](engineering-principles.md)'s `## Error handling` section; this note covers what errors and logs must look like once they exist. Each stack's `error-handling.md` only adds its own mechanism (handler, logging library, where the correlation ID is wired) and never contradicts this note.

## Exception handling

- Failures are caught and mapped in **one central place** (the stack's global handler/interceptor/boundary) — never try/catch scattered per endpoint, screen, or component.
- Domain/application errors are specific types, never a generic exception caught and stringly-typed.
- Any error surfaced to a caller (API response, exception message reaching a client or log) carries a stable, predictable code (e.g. `ORDER_NOT_FOUND`, never a raw exception message) plus a human-readable message with relevant context (which order, which field) — never a generic "something went wrong".
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable message.
- Never include sensitive data (passwords, tokens, secrets, full card numbers) in an error message, log entry, or exception payload — not even at debug level.

## Logging

- Every service/app can answer "what happened to this request" without attaching a debugger — leveled, structured logging through the stack's logging library, never bare `print`/`console.log` scattered through the code.
- Levels: `error` for failures that need attention (unhandled exceptions, failed external calls), `warn` for recoverable/unexpected situations (a retried request, falling back to a default), `info` for business-relevant events (order created, user logged in), `debug` for development-only detail. Never let `debug`/`info` volume drown out `error`/`warn` in production.
- Structured logging always: named fields (message templates, MDC, parameterized messages), never string-interpolated messages — fields need to stay queryable in the log sink, not buried in a formatted string.
- The minimum level is configured per environment; on client apps, `debug`/`info` output is stripped or no-op'd in release/production builds and never shipped to a real user's device.

## Correlation ID

The `X-Correlation-Id` header is the contract that lets one failure be traced end-to-end, client to backend.

- **Backend**: every request carries `X-Correlation-Id`. If the client doesn't send one, the backend generates it early in the request pipeline and includes it in the response, so the caller can reuse it for related/retried requests. It's attached to the log context for the lifetime of the request, so every log line emitted while handling it — including from any downstream service it calls — traces back to the same request.
- **Client**: attaches the current `X-Correlation-Id` to every outgoing request if one is already known for the ongoing user flow (e.g. carried over from a previous response for a multi-step operation); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side log line related to that request, and reused for an immediate retry of the same logical operation.

## Crash reporting

No crash-reporting service (Sentry, Firebase Crashlytics) is wired up yet — the stack's global boundary logs locally and shows fallback UI. This is a deliberate current scope, not an oversight.
