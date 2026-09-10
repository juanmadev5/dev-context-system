---
tags: [aspnet-core, testing]
---

# Testing — ASP.NET Core

Universal testing rules for every ASP.NET Core project. [coding-standards](coding-standards.md)'s `## Testing` section decides *when* a test is mandatory; this note covers *how* one is written once it's warranted.

## Stack

- **xUnit** as the test runner, **Moq** for mocking, **FluentAssertions** for readable assertions (`result.Should().Be(...)` instead of a bare `Assert.Equal`).
- **Testcontainers** (`Testcontainers.PostgreSql`, `Testcontainers.Redis`) for integration tests that need a real Postgres/Redis — never the shared local instance from [local-infrastructure](../../04-infra/local-infrastructure.md), which stays reserved for manual/dev use. Each integration test class spins up its own ephemeral container, so tests stay isolated and reproducible on any machine, CI included, with no state shared between runs.

## Structure — Given-When-Then

- Test method names read as `Given_<context>_When_<action>_Then_<outcome>` (e.g. `Given_OrderHasNoItems_When_Processing_Then_ThrowsInvalidOperationException`).
- The body is split into three commented blocks in the same order, replacing the generic Arrange/Act/Assert labels:

```csharp
[Fact]
public async Task Given_OrderHasNoItems_When_Processing_Then_ThrowsInvalidOperationException()
{
    // Given
    var order = new Order(items: []);
    var sut = new OrderService(_repositoryMock.Object);

    // When
    var act = () => sut.ProcessOrderAsync(order);

    // Then
    await act.Should().ThrowAsync<InvalidOperationException>();
}
```

- One `[Fact]`/`[Theory]` per scenario — don't cram multiple Given/When combinations and several unrelated asserts into a single test.

## Mocking policy

- **Mock every injected dependency in a unit test** — repositories, external clients, clock/time providers, everything the class under test receives via constructor injection. A unit test never touches a real database, network call, or filesystem.
- Integration tests are the ones that exercise real infrastructure (via Testcontainers above) — they trade the isolation of mocks for confidence that the actual EF Core mapping and DbUp-applied schema behave correctly against real Postgres. Don't mix the two styles in the same test class.

## What to test

- Same judgment call as [coding-standards](coding-standards.md)'s `## Testing` section: not a blanket requirement, mandatory for delicate business logic (money, permissions, state transitions, concurrency).
- Prefer testing through the public API of a class/use case, not its private implementation details — a test that breaks every time an internal helper is renamed, without the observable behavior changing, is testing the wrong thing.

## Sources

Record any testing-library API verified against official docs in `docs/SOURCES.md` per [sources](sources.md).
