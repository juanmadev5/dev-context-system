---
tags: [vuejs, error-handling, logging]
---

# Error Handling & Logging — Vue.js

Universal rules for error handling and logging in every Vue project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — the API client's error normalization, the global error handler, and structured logging.

## Exception handling

- A single, shared API client wrapper (an `axios` instance with a response interceptor, or a `fetch` wrapper used by every composable in `services/`) is the one place raw HTTP errors get caught and normalized into a common `AppError` type before they reach a composable/component — feature code consumes `AppError`, never a raw `AxiosError`/`Response` deep in a composable.
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable message via the normalized `AppError`.

## Global error boundary

- **`app.config.errorHandler`** (registered once at app creation) catches any error Vue's own render/watcher machinery doesn't otherwise handle, logs it, and routes the user to a fallback state (a dedicated error UI, not a frozen/blank screen).
- A top-level component additionally uses **`onErrorCaptured`** to stop a child-tree error from propagating further than necessary when a more local fallback (e.g. just that widget/section) is enough, instead of tearing down the whole app for a failure contained to one feature.
- No crash-reporting service (Sentry) wired up yet — the handler above logs locally and shows fallback UI; this is a deliberate current scope, not an oversight.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).
- Never log secrets, tokens, or full sensitive identifiers (passwords, card numbers) — not even at `debug` level.

## Correlation ID

- The API client wrapper attaches the current `X-Correlation-Id` to every outgoing request if one is already known for the ongoing user flow (carried over from a previous response); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side log line related to that request, and reused for an immediate retry of the same logical operation, so a failure can be traced end-to-end through the backend's own logs.
