---
tags: [jetpack-compose, error-handling, logging]
---

# Error Handling & Logging — Jetpack Compose

Universal rules for error handling and logging in every Jetpack Compose project. [coding-standards](coding-standards.md)'s `## Error handling` section covers boundary validation; this note covers the mechanism — exception mapping, the global error boundary, and structured logging.

## Exception handling

- The **data layer** (repositories/data sources in `data/`, per [architecture-principles](architecture-principles.md)) is where raw platform/network exceptions get caught and mapped into typed domain exceptions — a ViewModel never catches a raw `IOException`/Retrofit exception directly.
- A ViewModel that calls a use case/repository and gets a domain exception maps it into an error UI state (part of the single immutable `StateFlow` it exposes, per [jetpack-compose.md](jetpack-compose.md)'s MVVM rules) — the Composable reads that state and renders the corresponding UI, it never catches exceptions itself.
- Never surface a raw exception message or stack trace in end-user-facing UI — map it to a stable, human-readable string resource ([jetpack-compose.md](jetpack-compose.md)'s no-hardcoded-strings rule applies here too) via the same error state.

## Global error boundary

- Uncaught exceptions are caught at the process level via a custom `Thread.UncaughtExceptionHandler` (installed at app startup) that logs the error before the default system crash dialog takes over — this is what makes "what killed the app" visible in logs instead of only in the OS's own crash report.
- Within Compose itself, a top-level error boundary Composable wraps the app's content and shows a fallback screen when a caught, unrecoverable state is reached (as opposed to relying on Compose to keep rendering a broken tree) — paired with the ViewModel-level error-state mapping above for anything that's actually recoverable.
- No crash-reporting service (Firebase Crashlytics, Sentry) wired up yet — the boundary above logs locally and shows fallback UI; this is a deliberate current scope, not an oversight.

## Logging

- **Timber** for all logging, not the raw `android.util.Log` — auto-derives the tag from the calling class and makes it trivial to plant a no-op tree in release builds instead of shipping debug logs to production.
- Log levels: `e` (error) for failures that need attention (unhandled exceptions, failed API calls), `w` (warn) for recoverable/unexpected situations, `i` (info) for business-relevant events (user logged in, purchase completed), `d`/`v` (debug/verbose) for development-only detail — a release-build `Timber.Tree` no-ops `d`/`v`/`i`, only `w`/`e` ship (adjust per project if `i` is genuinely needed for release diagnostics).
- Never log secrets, tokens, or full sensitive identifiers (passwords, card numbers) — not even at debug/verbose level.

## Correlation ID

- Every outgoing API request attaches the current `X-Correlation-Id` if one is already known for the ongoing user flow (e.g. carried over from a previous response for a multi-step operation); otherwise it's left for the backend to generate. The response's `X-Correlation-Id` is read and included in any client-side Timber log line related to that request, and reused for an immediate retry of the same logical operation, so a failure can be traced end-to-end through the backend's own logs.
