---
tags: [react, error-handling, logging]
---

# Error Handling & Logging — React

Universal rules for error handling and logging in every React project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — the API client's error normalization, the global error boundary, and structured logging.

## Exception handling

- A single, shared API client wrapper (an `axios` instance with a response interceptor, or a `fetch` wrapper) is the one place raw HTTP errors get caught and normalized into a common `AppError` type before they reach a hook/component — feature code consumes `AppError`, never a raw `AxiosError`/`Response` deep in a hook.
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable message via the normalized `AppError`.

## Global error boundary

- A single top-level **React Error Boundary** component (`componentDidCatch`/`static getDerivedStateFromError` — there's no hook equivalent, so this stays a class component even in an otherwise all-function-component codebase) wraps the app and renders a fallback UI for any uncaught render-time error — the app never shows a blank white screen.
- The boundary only catches render/lifecycle errors, not errors inside event handlers or async code — those are caught at the source (the API client wrapper above, or a local try/catch around the async call) and turned into UI state, not left to bubble up expecting the boundary to catch them.
- No crash-reporting service (Sentry) wired up yet — the boundary above logs locally and shows fallback UI; this is a deliberate current scope, not an oversight.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).
- Never log secrets, tokens, or full sensitive identifiers (passwords, card numbers) — not even at `debug` level.

## Correlation ID

- The API client wrapper attaches the current `X-Correlation-Id` to every outgoing request if one is already known for the ongoing user flow (carried over from a previous response); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side log line related to that request, and reused for an immediate retry of the same logical operation, so a failure can be traced end-to-end through the backend's own logs.
