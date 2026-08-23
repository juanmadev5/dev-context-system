---
tags: [infra, deployment, vercel, docker]
---

# Deployments

## Web frontends

- Deployed to **Vercel** — see [[00-global/tech-stack-map|tech-stack-map]]. Applies to [[02-web/angular/angular|angular]], [[02-web/vuejs/vuejs|vuejs]], [[02-web/react/react|react]] projects.
- Local dev for these still runs through [[04-infra/docker|docker]] where the project has supporting services to talk to (e.g. a local [[03-backend/aspnet-core/aspnet-core|aspnet-core]]/[[03-backend/spring-boot/spring-boot|spring-boot]] backend, [[04-infra/local-infrastructure|local-infrastructure]]).
- Environment variables/secrets configured in Vercel's project settings, never committed to the repo.

## Backend

- Deployed as a **Docker** container in production, same as in dev — see [[04-infra/docker|docker]]. No Vercel/serverless deployment for the ASP.NET Core or Spring Boot backend.
- Prefer a deploy pipeline that builds the image from the same multi-stage Dockerfile used locally, so dev/prod parity holds.

## See also

- [[04-infra/docker|docker]], [[03-backend/aspnet-core/aspnet-core|aspnet-core]], [[03-backend/spring-boot/spring-boot|spring-boot]]
