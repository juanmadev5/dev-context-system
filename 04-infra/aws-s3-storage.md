---
tags: [infra, storage, aws, s3]
---

# AWS S3 Storage

File storage when the project has a dedicated [[aspnet-core]] or [[spring-boot]] backend — see [[tech-stack-map]]. Use [[supabase-storage]] instead for Supabase-only projects.

## Conventions

- The backend mediates all access: clients get pre-signed URLs for upload/download rather than holding long-lived AWS credentials directly.
- Bucket names, key prefixes/paths, and content-type allowlists: constants, never magic strings, per [[coding-standards]].
- Object keys are structured and predictable (e.g. `{tenant-or-user-id}/{resource-type}/{id}/{filename}`), not random/opaque, so cleanup and access-scoping stay tractable.
- Bucket policies default to private; public access is opt-in per bucket/prefix and deliberate, never the default.

## See also

- [[aspnet-core]], [[spring-boot]]
- [[supabase-storage]]
