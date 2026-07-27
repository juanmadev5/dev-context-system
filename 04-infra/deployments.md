---
tags: [infra, deployment, vercel, docker]
---

# Deployments

## Web frontends

- Deployed to **Vercel** — see [[tech-stack-map]]. Applies to [[angular]], [[vuejs]], [[react]], [[astro]], [[html-tailwind]] projects.
- Local dev for these still runs through [[docker]] where the project has supporting services to talk to (e.g. a local [[aspnet-core]]/[[spring-boot]] backend, [[local-infrastructure]]).
- Environment variables/secrets configured in Vercel's project settings, never committed to the repo.

## Backend

- Deployed as a **Docker** container in production, same as in dev — see [[docker]]. No Vercel/serverless deployment for the ASP.NET Core or Spring Boot backend.
- Prefer a deploy pipeline that builds the image from the same multi-stage Dockerfile used locally, so dev/prod parity holds.

## Supabase-backed projects

- No separate backend deployment — see [[supabase]]. Only the frontend needs a deployment target (Vercel).

## See also

- [[docker]], [[aspnet-core]], [[spring-boot]], [[supabase]]
