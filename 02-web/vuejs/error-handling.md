---
tags: [vuejs, error-handling]
---

# Error Handling & Logging — Vue.js

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: the API client's error normalization, the global error handler, and structured logging.

## Exception handling

- A single, shared API client wrapper (an `axios` instance with a response interceptor, or a `fetch` wrapper used by every composable in `services/`) is the one place raw HTTP errors get caught and normalized into a common `AppError` type before they reach a composable/component — feature code consumes `AppError`, never a raw `AxiosError`/`Response` deep in a composable.

## Global error boundary

- **`app.config.errorHandler`** (registered once at app creation) catches any error Vue's own render/watcher machinery doesn't otherwise handle, logs it, and routes the user to a fallback state (a dedicated error UI, not a frozen/blank screen).
- A top-level component additionally uses **`onErrorCaptured`** to stop a child-tree error from propagating further than necessary when a more local fallback (e.g. just that widget/section) is enough, instead of tearing down the whole app for a failure contained to one feature.

## Logging

- **loglevel** for all logging — a leveled wrapper (`error`/`warn`/`info`/`debug`) around `console.*` instead of bare `console.log` calls scattered through the codebase; the minimum log level is configured per environment (`debug`/`info` enabled in development, `warn`/`error` only in production builds).

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Client side of the contract, implemented in the API client wrapper.
