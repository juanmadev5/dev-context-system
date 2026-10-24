---
tags: [spring-boot, testing]
---

# Testing — Spring Boot

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds Spring Boot's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **JUnit 5 + Mockito** (`spring-boot-starter-test` bundles both, plus AssertJ for readable assertions — `assertThat(result).isEqualTo(...)`).
- **Testcontainers** for integration tests that need a real Postgres/Redis — never a shared manual-dev instance. Postgres uses the dedicated `testcontainers-postgresql` module; Redis has no dedicated official module, so it's a plain `GenericContainer` pointed at a `redis` image. Each integration test class spins up its own ephemeral container via `@Testcontainers` + `@Container`, so tests stay isolated and reproducible on any machine, CI included, with no state shared between runs.

## Structure — Given-When-Then

- Test method names read as `given<Context>_when<Action>_then<Outcome>` (e.g. `given_orderHasNoItems_when_processing_then_throwsIllegalStateException`).

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

## Mocking

- Constructor-injected dependencies are mocked with `@Mock` + `@InjectMocks`, or `Mockito.mock(...)`. Integration tests (Testcontainers above, typically with `@SpringBootTest`) verify the actual Spring Data JPA mapping and Flyway-applied schema against real Postgres.
