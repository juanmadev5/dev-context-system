---
tags: [global, dependencies]
---

# Dependency Management

Applies to every project, regardless of stack — how a dependency is chosen, versioned, and kept up to date.

## Before adding a new dependency

- **Check an existing dependency doesn't already solve it.** Adding a second library for something already covered by one already installed is duplication at the dependency level, not just the code level.
- **Maintenance is active.** Recent commits/releases, issues actually getting responses — not a project that's gone quiet for a year or more.
- **License is not commercial/paid.** Open-source licenses (MIT, Apache-2.0, BSD, MPL) are fine by default. A dependency that requires a commercial license or per-seat/per-usage fee is never added without explicit approval first — this is a business decision, not an engineering one.
- If a dependency fails any of these, it's a question for the developer, not a judgment call to make silently.

## Versioning

- **Caret/tilde ranges are allowed for minor/patch versions** (`^1.2.3` in `package.json`, the ecosystem-equivalent floating minor/patch elsewhere) — never a range that allows a major version bump, since that's where breaking changes live.
- The **lockfile is what actually pins the resolved version** (`pnpm-lock.yaml`, `packages.lock.json`, a Maven `dependency:tree`-verified `pom.xml`) and is always committed — a caret range in the manifest is a stated tolerance for what's allowed to move, not a substitute for reproducible installs.
- Bumping a major version is a deliberate, reviewed change (read the changelog, check for breaking changes called out) — never an incidental side effect of a routine `pnpm update`/restore.

## Keeping dependencies current

- Vulnerability/staleness checks use each ecosystem's own native tooling — `pnpm audit`, `dotnet list package --vulnerable`, `mvn org.owasp:dependency-check:check` — run manually or wired into CI. No centralized bot/service (Snyk, Dependabot, Renovate) across projects at this point; that's a deliberate choice, not an oversight, and can change per-project if a specific project's context calls for it.
