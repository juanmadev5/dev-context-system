---
tags: [infra, storage, aws, s3]
---

# AWS S3 Storage

File storage for every project — see [[00-global/tech-stack-map|tech-stack-map]].

## Conventions

- The backend mediates all access: clients get pre-signed URLs for upload/download rather than holding long-lived AWS credentials directly.
- Bucket names, key prefixes/paths, and content-type allowlists: constants, never magic strings, per the project's stack's own no-magic-values rule.
- Object keys are structured and predictable (e.g. `{tenant-or-user-id}/{resource-type}/{id}/{filename}`), not random/opaque, so cleanup and access-scoping stay tractable.
- Bucket policies default to private; public access is opt-in per bucket/prefix and deliberate, never the default.

## See also

- [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]
