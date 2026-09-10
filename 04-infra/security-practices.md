---
tags: [infra, security]
---

# Security Practices

Practical security conventions for every project, independent of stack — the standing practices that hold before a code review even happens.

## Secrets

- Secrets (credentials, API keys, tokens, connection strings) are configured via **environment variables or mounted secrets at runtime**, never baked into an image or committed to the repository.
- No secret manager (Vault, AWS Secrets Manager, Doppler) in the current setup — env vars / mounted secrets is the whole mechanism for now. If a project's threat model genuinely needs one later, that's a deliberate, separate decision, not a default to reach for.
- A leaked or suspected-leaked secret is rotated immediately, not just removed from wherever it was found — the old value is treated as permanently compromised.

## Input validation & output encoding

- Every system boundary (HTTP request body/params, file uploads, messages from a queue) validates its input before acting on it — never trust that a client only sends what the UI allows it to send.
- Output that includes user-controlled content is encoded for the context it's rendered in (HTML-escaped in a web response, parameterized rather than concatenated in a query) — this is what prevents injection classes (XSS, SQL injection) from a validation gap turning into an actual exploit.

## Rate limiting

- Every backend exposes its own rate limiting for public endpoints, at minimum on abuse-sensitive ones (login, registration, password reset, anything expensive to compute) — not left entirely to an upstream proxy/CDN to handle.

## Transport & access

- HTTPS/TLS everywhere past local development — no plaintext HTTP between a client and a production service.
- Authorization is checked per resource/action on every request that needs it — authentication (who the caller is) is never treated as a substitute for authorization (what that caller is allowed to do).
- Service-to-service and database credentials follow least privilege — a service gets only the permissions/roles it actually needs, not a shared superuser/admin credential reused everywhere out of convenience.

## Logging

- Never log secrets, tokens, passwords, or full sensitive identifiers (card numbers, government IDs) — not even at debug level. This applies to application logs, error payloads, and any crash/error-reporting service equally.

## Dependency hygiene

- A dependency's maintenance status and known-vulnerability history are a security concern, not just a code-quality one — an unmaintained package with an open CVE is a standing liability even if nothing in the project's own code is wrong.
