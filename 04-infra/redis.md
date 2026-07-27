---
tags: [infra, cache, redis]
---

# Redis

Cache layer, used with [[aspnet-core]] or [[spring-boot]] backends — see [[tech-stack-map]]. Locally provided by [[local-infrastructure]] (AOF persistence enabled, port `6379` by default).

## Conventions

- Cache keys are always namespaced and built from constants, never hand-built inline strings scattered across the codebase — e.g. `cache:orders:{orderId}` built via a helper/constant prefix, per [[coding-standards]].
- Every cached entry has an explicit TTL appropriate to how stale the data is allowed to be — no un-expiring cache entries unless deliberately intended as a persistent store (which Redis is not the right tool for here; that's Postgres's job).
- Cache invalidation is explicit at the write path that changes the underlying data — don't rely on TTL alone when correctness matters.
- Redis here is a cache, not a system of record — don't store data in Redis that doesn't also exist (or can't be rebuilt) from [[postgresql]].

## See also

- [[local-infrastructure]], [[aspnet-core]], [[spring-boot]]
