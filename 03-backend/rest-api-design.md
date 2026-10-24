---
tags: [backend, api, rest]
---

# REST API Design

Used on every backend project that exposes a REST API — [aspnet-core](aspnet-core/aspnet-core.md), [spring-boot](spring-boot/spring-boot.md) — see [tech-stack-map](../00-global/tech-stack-map.md). Not a stack of its own: a project copies this note directly, alongside whichever backend stack's own folder it uses.

## Resources, not actions

- URLs represent resources (nouns), never actions (verbs). The HTTP method expresses what happens to the resource, not the path.
- Bad: `POST /createOrder`, `POST /orders/123/cancelOrder`
- Good: `POST /orders`, `DELETE /orders/123` (or `POST /orders/123/cancellation` if cancellation isn't deletion) — the verb is the HTTP method, the noun is the URL.

## Predictable URLs

- One consistent resource-naming convention across the whole API — plural nouns (`/orders`), never a mix of `/order` and `/orders`.
- A consistent hierarchy: collection → identifier → sub-resource — `/orders/{orderId}/items/{itemId}`. Never skip a level or reorder it differently elsewhere in the same API.

## HTTP methods mean what they say

- `GET` — read, no side effects, safe to retry and to cache.
- `POST` — create a new resource (or trigger a non-idempotent action modeled as a sub-resource).
- `PUT` — replace a resource in full.
- `PATCH` — partial update.
- `DELETE` — remove a resource.
- Idempotency matters: a client retrying a `PUT`/`DELETE` after a dropped connection must be safe to repeat without a different outcome. Don't use `GET` or `PUT` for anything that isn't actually idempotent.

## Status codes carry meaning

- Don't return `200 OK` for everything and put the real result in the body. Use the status code itself:
  - `200 OK` — successful read/update.
  - `201 Created` — successful creation, with a `Location` header pointing at the new resource.
  - `204 No Content` — successful action with nothing to return (e.g. a `DELETE`).
  - `400 Bad Request` — malformed request.
  - `401 Unauthorized` / `403 Forbidden` — missing/insufficient auth.
  - `404 Not Found` — resource doesn't exist.
  - `409 Conflict` — request conflicts with current state (e.g. duplicate creation).
  - `422 Unprocessable Entity` — well-formed request, semantically invalid (validation errors).
  - `500 Internal Server Error` — unhandled server-side failure.
- A correct status code makes a custom error-handling protocol on top of HTTP unnecessary.

## Consistent error responses

- Every error response — regardless of endpoint — follows the same shape: a stable machine-readable code, a human-readable message, and, for validation errors, which field(s) failed and why.

```json
{
  "code": "VALIDATION_FAILED",
  "message": "The request contains invalid fields.",
  "errors": [
    { "field": "email", "message": "must be a valid email address" }
  ]
}
```

- Same principle as the error-handling rule in [engineering-principles](../00-global/engineering-principles.md), applied to the wire format: a predictable shape lets a client branch on `code` instead of parsing `message`.

## Query parameters, not path segments, for anything optional

- The path identifies *which* resource. Filtering, sorting, pagination, and any optional criteria go in query parameters, never encoded into the path.
- Bad: `/orders/status/pending/sortby/date`
- Good: `/orders?status=pending&sort_by=date`

## Pagination and filtering, no exceptions

**Every endpoint that returns a collection must be paginated. No exceptions** — a list that "currently" returns few items is still a list, and unpaginated collections are a scaling and abuse liability from day one, not something to retrofit later.

- Query params: `page` (1-based, default `1`) and `page_size` (project-defined default and hard max — e.g. default `20`, max `100` — never unbounded; a client can't request `page_size=999999`).
- Paginated responses are always wrapped in an envelope, never a bare array, so metadata travels with the data — the same shape across every stack in this repository, regardless of backend:

  ```json
  {
    "items": [ ... ],
    "page": 1,
    "page_size": 20,
    "total_count": 137,
    "total_pages": 7
  }
  ```

- Filters are query params named after the field they filter (e.g. `GET /orders?status=Active&created_after=2026-01-01`), applied server-side *before* pagination — never a single opaque "filter" blob param unless the filtering surface is genuinely generic/dynamic enough to need one.
- The envelope type, default/max page size constants, and query-param names are defined once per project and reused by every paginated endpoint — never redefined per feature, per [engineering-principles](../00-global/engineering-principles.md)'s DRY rule.

## Documented via OpenAPI, no exceptions

**Every endpoint must be documented via OpenAPI — no exceptions.** The generated spec is what a frontend/mobile consumer or an API client generator actually reads — an undocumented endpoint is a broken contract, not a cosmetic gap.

- Every endpoint documents a summary/description and every possible response status it can actually return — success and error alike, not just the happy path.
- Non-trivial DTO properties (units, format, constraints) get a description too, but not mechanically on every property — same restraint as [engineering-principles](../00-global/engineering-principles.md)'s comment guidance.
- The concrete mechanism (library, annotations/attributes, UI served) is stack-specific — see that stack's own note.

## Versioned via the URL path, no exceptions

**Every endpoint is versioned via the URL path (`/api/v1/...`) — no exceptions.** A "v1 for now, we'll version later if we need to" mindset is how a breaking change ends up shipped straight to every existing client instead of landing behind a new version segment.

- URL path segment only (`/api/v1/...`) — never a query-string (`?api-version=1`) or header-based (`X-Api-Version`) scheme as the primary mechanism; the version needs to be visible in the URL a developer pastes into a browser or shares in a bug report, not hidden in a header.
- Additive changes (new optional field, new endpoint) don't need a version bump — they ship on the existing version.
- Bump the major segment (`v1` → `v2`) only for a breaking change (removed/renamed field, changed status code, changed semantics) — with the old version kept alive until clients have migrated. Never silently break an existing endpoint's contract.
- The concrete mechanism (versioning library, route template, attribute/annotation) is stack-specific — see that stack's own note.

## Consistent formats end to end

- `snake_case` everywhere on the wire — request body fields, response body fields, and query parameter names alike, on every endpoint, no exceptions. Never `camelCase` or a mix of the two.
- One date/time format (ISO 8601, UTC) across every endpoint.
- One response envelope shape (or deliberately none) applied consistently — a client that has learned one endpoint should be able to guess the shape of every other one.
