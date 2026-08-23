---
tags: [infra, cache, redis]
---

# Redis

Cache layer, used with [[03-backend/aspnet-core/aspnet-core|aspnet-core]] or [[03-backend/spring-boot/spring-boot|spring-boot]] backends — see [[00-global/tech-stack-map|tech-stack-map]]. Locally provided by [[04-infra/local-infrastructure|local-infrastructure]] (AOF persistence enabled, port `6379` by default).

## Conventions

- Cache keys are always namespaced and built from constants, never hand-built inline strings scattered across the codebase — e.g. `cache:orders:{orderId}` built via a helper/constant prefix, per the project's stack's own no-magic-values rule.
- Every cached entry has an explicit TTL appropriate to how stale the data is allowed to be — no un-expiring cache entries unless deliberately intended as a persistent store (which Redis is not the right tool for here; that's Postgres's job).
- Cache invalidation is explicit at the write path that changes the underlying data — don't rely on TTL alone when correctness matters.
- Redis here is a cache, not a system of record — don't store data in Redis that doesn't also exist (or can't be rebuilt) from [[04-infra/postgresql|postgresql]].

## See also

- [[04-infra/local-infrastructure|local-infrastructure]], [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]
