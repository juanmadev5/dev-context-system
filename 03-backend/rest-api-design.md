---
tags: [backend, api, rest]
---

# REST API Design

Used on every backend project that exposes a REST API — [aspnet-core](aspnet-core/aspnet-core.md), [spring-boot](spring-boot/spring-boot.md) — see [tech-stack-map](../00-global/tech-stack-map.md). Not a stack of its own: a project imports this note directly, alongside whichever backend stack's `INDEX.md` it uses.

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

- Same principle as the stack's own coding-standards error-handling rule (see [aspnet-core's coding-standards](aspnet-core/coding-standards.md) or [spring-boot's coding-standards](spring-boot/coding-standards.md)), applied to the wire format: a predictable shape lets a client branch on `code` instead of parsing `message`.

## Query parameters, not path segments, for anything optional

- The path identifies *which* resource. Filtering, sorting, pagination, and any optional criteria go in query parameters, never encoded into the path.
- Bad: `/orders/status/pending/sortby/date`
- Good: `/orders?status=pending&sortBy=date`

## Version breaking changes deliberately

- Additive changes (new optional field, new endpoint) don't need a version bump.
- A breaking change (removing/renaming a field, changing a type, changing semantics) needs an explicit versioning strategy — a version segment in the URL (`/v2/orders`) or a version header — with the old version kept alive until clients have migrated. Never silently break an existing endpoint's contract.

## Consistent formats end to end

- One casing convention for JSON field names (`camelCase` or `snake_case`, pick one) used by every endpoint, no exceptions.
- One date/time format (ISO 8601, UTC) across every endpoint.
- One response envelope shape (or deliberately none) applied consistently — a client that has learned one endpoint should be able to guess the shape of every other one.
