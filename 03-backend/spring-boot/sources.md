---
tags: [spring-boot, documentation, sources]
---

# Sources — Spring Boot

Curated index of this stack's own official documentation — the vetted entry points to consult first, instead of rediscovering them from scratch on every project. Every link below points to the vendor/maintainer's own official docs (or the relevant RFC/spec).

## Official documentation only

- **Only the source's own official documentation** — the vendor/maintainer's docs site, the package's own repo (README, wiki, official guide), or a relevant standard/spec.
- **Never** blogs, Medium/dev.to posts, Stack Overflow, random tutorials, or AI-generated summary/aggregator sites — even if one of those turns up first in a search.
- For this stack, official means: `docs.spring.io` (Spring Boot, Spring Data, Spring Security, Spring Framework) and each library's own docs domain listed below.

## Spring Boot

- Official reference documentation entry point: https://docs.spring.io/spring-boot/index.html
- Maven plugin reference (packaging, running, build-info generation): https://docs.spring.io/spring-boot/maven-plugin/

## Spring Data JPA

- Reference docs for repository interfaces, query methods, and `JpaRepository`: https://docs.spring.io/spring-data/jpa/reference/

## Spring Security

- OAuth2 Resource Server reference (JWT/opaque bearer token validation, the Keycloak pairing): https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html

## Spring Data Redis / Caching

- Spring Data Redis reference docs: https://docs.spring.io/spring-data/redis/reference/
- Spring Boot caching reference (`@Cacheable`, cache provider auto-detection incl. Redis): https://docs.spring.io/spring-boot/reference/io/caching.html

## Bean Validation / Jakarta Validation

- Spring Framework's Bean Validation integration guide (`@Valid`, `LocalValidatorFactoryBean`, method validation): https://docs.spring.io/spring-framework/reference/core/validation/beanvalidation.html
- Jakarta Bean Validation 3.0 specification: https://jakarta.ee/specifications/bean-validation/3.0/

## Flyway

- Official Redgate Flyway documentation (migrations, versioned scripts, schema history): https://documentation.red-gate.com/flyway

## springdoc-openapi

- Official documentation (OpenAPI 3 + Swagger UI for Spring Boot): https://springdoc.org/

## JUnit 5

- Official JUnit user guide (Jupiter, Platform, Vintage): https://docs.junit.org/current/user-guide/

## Mockito

- Official Mockito site: https://site.mockito.org/

## SLF4J + Logback

- SLF4J user manual: https://www.slf4j.org/manual.html
- Logback manual: https://logback.qos.ch/manual/index.html

## OpenTelemetry (Java)

- Java documentation (traces, metrics, logs): https://opentelemetry.io/docs/languages/java/

## Checkstyle

- Official Checkstyle documentation (checks, configuration, rules): https://checkstyle.sourceforge.io/
- Apache Maven Checkstyle Plugin documentation: https://maven.apache.org/plugins/maven-checkstyle-plugin/

## RFC 7807

- Problem Details for HTTP APIs — note: obsoleted by RFC 9457, worth checking if the newer RFC should be adopted instead: https://www.rfc-editor.org/rfc/rfc7807

## If a source isn't listed here

This index covers the libraries/topics this vault's conventions actually name — it isn't exhaustive. If a task needs an official documentation source that isn't listed above:

- Look it up directly — same rule applies: only the vendor/maintainer's own official docs, never a blog/tutorial/Stack Overflow/AI-summary site.
- Record it in that project's own `docs/SOURCES.md` (created the first time it's actually needed, per [[03-backend/spring-boot/coding-standards|coding-standards]]'s "no empty folders" reasoning applied to documentation), grouped by library under a `##` heading, one bullet per source:

  ```markdown
  ## SomeNewLibrary

  - What the page covers: https://...
  ```

## See also

- [[03-backend/spring-boot/coding-standards|coding-standards]]
- [[00-global/readme-conventions|readme-conventions]]
