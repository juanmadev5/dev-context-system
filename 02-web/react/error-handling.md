---
tags: [react, error-handling]
---

# Error Handling & Logging — React

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: the API client's error normalization, the global error boundary, and structured logging.

## Exception handling

- A single, shared API client wrapper (an `axios` instance with a response interceptor, or a `fetch` wrapper) is the one place raw HTTP errors get caught and normalized into a common `AppError` type before they reach a hook/component — feature code consumes `AppError`, never a raw `AxiosError`/`Response` deep in a hook.

## Global error boundary

- A single top-level **React Error Boundary** component (`componentDidCatch`/`static getDerivedStateFromError` — there's no hook equivalent, so this stays a class component even in an otherwise all-function-component codebase) wraps the app and renders a fallback UI for any uncaught render-time error — the app never shows a blank white screen.
- The boundary only catches render/lifecycle errors, not errors inside event handlers or async code — those are caught at the source (the API client wrapper above, or a local try/catch around the async call) and turned into UI state, not left to bubble up expecting the boundary to catch them.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Client side of the contract, implemented in the API client wrapper.
