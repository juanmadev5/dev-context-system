---
tags: [infra, storage, supabase]
---

# Supabase Storage

File storage for Supabase-only projects — see [[tech-stack-map]], [[supabase]]. Use [[aws-s3-storage]] instead once the project has a dedicated [[aspnet-core]] or [[spring-boot]] backend.

## Conventions

- Bucket-level access controlled via Storage RLS policies, mirroring the same authorization approach as the rest of [[supabase]] — never rely on "security through obscurity" object paths as the only protection for private buckets.
- Bucket names and path structure: constants, never magic strings, per [[coding-standards]].
- Public buckets are opt-in and deliberate; default to private.

## See also

- [[supabase]], [[supabase-auth]]
- [[aws-s3-storage]]
