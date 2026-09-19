---
tags: [flutter, error-handling]
---

# Error Handling & Logging — Flutter

Stack-specific additions to [error-handling-principles](../../00-global/error-handling-principles.md) — read that first; this note covers the mechanism here: exception mapping, the global error boundary, and structured logging.

## Exception handling

- The **data layer** (services/repositories in `data/`, per [architecture-principles](architecture-principles.md)) is where raw platform/HTTP/`dio`-or-`http`-package exceptions get caught and mapped into typed domain exceptions — a Bloc/Cubit never catches a raw `DioException`/`SocketException` directly.
- A Bloc/Cubit that calls a use case/repository and gets a domain exception maps it into an error state (a sealed state class carrying the failure, per [engineering-principles](../../00-global/engineering-principles.md)'s no-magic-values/enum rules) — the View reads that state and renders the corresponding UI, it never catches exceptions itself.

## Global error boundary

- **`FlutterError.onError`** (for framework/widget-build errors) and **`PlatformDispatcher.instance.onError`** (for uncaught async errors outside the widget tree) are both wired at app startup to log the error and show a fallback UI — the app never shows a blank/frozen screen or crashes silently.
- **`ErrorWidget.builder`** is overridden to render a fallback widget (not the default red error box) for any widget that fails to build, in release builds — the default debug-only error box stays for local development.

## Logging

- **`logger`** package for all logging — leveled output instead of scattering `print()`/`debugPrint()` calls through the codebase.
- Levels: `error`/`warning`/`info`/`debug`; `debug`/`info` output is stripped or no-op'd in release builds (a no-op `logger` output/filter swapped in for release).

## Correlation ID

Follows the `X-Correlation-Id` contract in [error-handling-principles](../../00-global/error-handling-principles.md).

- Client side of the contract, applied to every outgoing API request.
