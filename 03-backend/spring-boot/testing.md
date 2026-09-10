---
tags: [spring-boot, testing]
---

# Testing — Spring Boot

Universal testing rules for every Spring Boot project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

## Stack

- **JUnit 5 + Mockito** (`spring-boot-starter-test` bundles both, plus AssertJ for readable assertions — `assertThat(result).isEqualTo(...)`).
- **Testcontainers** (`testcontainers-postgresql`, module for Redis) for integration tests that need a real Postgres/Redis — never the shared local instance from [local-infrastructure](../../04-infra/local-infrastructure.md), which stays reserved for manual/dev use. Each integration test class spins up its own ephemeral container via `@Testcontainers` + `@Container`, so tests stay isolated and reproducible on any machine, CI included, with no state shared between runs.

## Structure — Given-When-Then

- Test method names read as `given<Context>_when<Action>_then<Outcome>` (e.g. `given_orderHasNoItems_when_processing_then_throwsIllegalStateException`).
- The body is split into three comment blocks in the same order, replacing the generic Arrange/Act/Assert labels:

```java
@Test
void given_orderHasNoItems_when_processing_then_throwsIllegalStateException() {
    // Given
    var order = new Order(List.of());
    var sut = new OrderService(repositoryMock);

    // When
    ThrowingCallable act = () -> sut.processOrder(order);

    // Then
    assertThatThrownBy(act).isInstanceOf(IllegalStateException.class);
}
```

- One `@Test` per scenario — don't cram multiple Given/When combinations and several unrelated asserts into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — repositories, external clients, clock/time providers, everything the class under test receives via constructor injection (`@Mock` + `@InjectMocks`, or `Mockito.mock(...)`). A unit test never touches a real database, network call, or filesystem.
- Integration tests are the ones that exercise real infrastructure (via Testcontainers above, typically with `@SpringBootTest`) — they trade the isolation of mocks for confidence that the actual Spring Data JPA mapping and Flyway-applied schema behave correctly against real Postgres. Don't mix the two styles in the same test class.

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a class/use case, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
