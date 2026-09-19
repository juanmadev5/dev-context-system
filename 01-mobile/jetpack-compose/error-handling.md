---
tags: [jetpack-compose, error-handling]
---

# Error Handling & Logging — Jetpack Compose

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: exception mapping, the global error boundary, and structured logging.

## Exception handling

- The **data layer** (repositories/data sources in `data/`, per [architecture-principles](architecture-principles.md)) is where raw platform/network exceptions get caught and mapped into typed domain exceptions — a ViewModel never catches a raw `IOException`/Retrofit exception directly.
- A ViewModel that calls a use case/repository and gets a domain exception maps it into an error UI state (part of the single immutable `StateFlow` it exposes, per [jetpack-compose.md](jetpack-compose.md)'s MVVM rules) — the Composable reads that state and renders the corresponding UI, it never catches exceptions itself.
- User-facing error text comes from a string resource ([jetpack-compose.md](jetpack-compose.md)'s no-hardcoded-strings rule applies here too), via the same error state.

## Global error boundary

- Uncaught exceptions are caught at the process level via a custom `Thread.UncaughtExceptionHandler` (installed at app startup) that logs the error before the default system crash dialog takes over — this is what makes "what killed the app" visible in logs instead of only in the OS's own crash report.
- Within Compose itself, a top-level error boundary Composable wraps the app's content and shows a fallback screen when a caught, unrecoverable state is reached (as opposed to relying on Compose to keep rendering a broken tree) — paired with the ViewModel-level error-state mapping above for anything that's actually recoverable.

## Logging

- **Timber** for all logging, not the raw `android.util.Log` — auto-derives the tag from the calling class and makes it trivial to plant a no-op tree in release builds instead of shipping debug logs to production.
- Levels: `e`/`w`/`i`/`d`/`v`; a release-build `Timber.Tree` no-ops `d`/`v`/`i`, only `w`/`e` ship (adjust per project if `i` is genuinely needed for release diagnostics).

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Client side of the contract, applied to every outgoing API request; client-side log lines related to a request go through Timber.
