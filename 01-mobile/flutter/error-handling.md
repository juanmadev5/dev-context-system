---
tags: [flutter, error-handling, logging]
---

# Error Handling & Logging — Flutter

Universal rules for error handling and logging in every Flutter project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — exception mapping, the global error boundary, and structured logging.

## Exception handling

- The **data layer** (services/repositories in `data/`, per [architecture-principles](architecture-principles.md)) is where raw platform/HTTP/`dio`-or-`http`-package exceptions get caught and mapped into typed domain exceptions — a Bloc/Cubit never catches a raw `DioException`/`SocketException` directly.
- A Bloc/Cubit that calls a use case/repository and gets a domain exception maps it into an error state (a sealed state class carrying the failure, per [coding-standards](coding-standards.md)'s no-magic-values/enum rules) — the View reads that state and renders the corresponding UI, it never catches exceptions itself.
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable message via the same error state.

## Global error boundary

- **`FlutterError.onError`** (for framework/widget-build errors) and **`PlatformDispatcher.instance.onError`** (for uncaught async errors outside the widget tree) are both wired at app startup to log the error and show a fallback UI — the app never shows a blank/frozen screen or crashes silently.
- **`ErrorWidget.builder`** is overridden to render a fallback widget (not the default red error box) for any widget that fails to build, in release builds — the default debug-only error box stays for local development.
- No crash-reporting service (Sentry, Firebase Crashlytics) wired up yet — the boundary above logs locally and shows fallback UI; this is a deliberate current scope, not an oversight.

## Logging

- **`logger`** package for all logging — leveled output instead of scattering `print()`/`debugPrint()` calls through the codebase.
- Log levels: `error` for failures that need attention (unhandled exceptions, failed API calls), `warning` for recoverable/unexpected situations, `info` for business-relevant events (user logged in, purchase completed), `debug` for development-only detail — `debug`/`info` output is stripped or no-op'd in release builds (a no-op `logger` output/filter swapped in for release), never shipped to a real user's device.
- Never log secrets, tokens, or full sensitive identifiers (passwords, card numbers) — not even at `debug` level.

## Correlation ID

- Every outgoing API request attaches the current `X-Correlation-Id` if one is already known for the ongoing user flow (e.g. carried over from a previous response for a multi-step operation); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side log line related to that request, and reused for an immediate retry of the same logical operation, so a failure can be traced end-to-end through the backend's own logs.
