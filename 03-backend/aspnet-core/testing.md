---
tags: [aspnet-core, testing]
---

# Testing — ASP.NET Core

Stack-specific additions to [testing-principles](../../00-global/testing-principles.md) — read that first; this note only adds ASP.NET Core's own libraries, naming format, and mocking mechanics and never contradicts it.

## Stack

- **xUnit** as the test runner, **Moq** for mocking, and xUnit's built-in `Assert` for assertions — no separate assertion library (FluentAssertions v8+ requires a commercial license, see [dependency-management](../../00-global/dependency-management.md)).
- **Testcontainers** (`Testcontainers.PostgreSql`, `Testcontainers.Redis`) for integration tests that need a real Postgres/Redis — never the shared local instance from [local-infrastructure](../../04-infra/local-infrastructure.md), which stays reserved for manual/dev use. Each integration test class spins up its own ephemeral container, so tests stay isolated and reproducible on any machine, CI included, with no state shared between runs.

## Structure — Given-When-Then

- Test method names read as `Given_<context>_When_<action>_Then_<outcome>` (e.g. `Given_OrderHasNoItems_When_Processing_Then_ThrowsInvalidOperationException`).

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
    await Assert.ThrowsAsync<InvalidOperationException>(act);
}
```

## Mocking

- Constructor-injected dependencies are mocked with Moq. Integration tests (Testcontainers above) verify the actual EF Core mapping and DbUp-applied schema against real Postgres.
