# .NET Coding Standards

These standards apply to all C# and .NET 8 projects within EWB. They are based on Microsoft's official C# coding conventions and Clean Architecture principles.

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Class | PascalCase | `OrderService` |
| Interface | PascalCase with `I` prefix | `IOrderRepository` |
| Method | PascalCase | `GetOrderById` |
| Property | PascalCase | `CreatedAt` |
| Private field | camelCase with `_` prefix | `_orderRepository` |
| Local variable | camelCase | `orderCount` |
| Constant | PascalCase | `MaxRetryCount` |
| Enum | PascalCase (type and values) | `OrderStatus.Pending` |
| Namespace | PascalCase, matching folder structure | `EWB.Payments.Application` |

---

## Project Structure

Follow Clean Architecture layer separation:

```
MyApp.API/            — Controllers, middleware, configuration, DI setup
MyApp.Application/    — Use cases, commands, queries, interfaces
MyApp.Domain/         — Entities, value objects, domain events, domain services
MyApp.Infrastructure/ — Database context, repositories, external services
MyApp.UnitTests/      — Tests for Application and Domain layers
MyApp.IntegrationTests/ — Tests for Infrastructure and API layers
```

Dependencies flow inward: API → Application → Domain. Infrastructure implements interfaces defined in Application.

---

## Classes and Methods

- One class per file; file name matches class name
- Keep classes focused — a class with more than 300 lines is a signal to review responsibility
- Prefer composition over inheritance
- Use `record` types for immutable data transfer objects
- Keep method bodies short — extract logic into well-named private methods when needed
- Avoid `static` methods except for pure utility functions with no dependencies

---

## Async/Await

- Use `async`/`await` throughout the call stack — do not mix synchronous and asynchronous code
- Always suffix async method names with `Async`: `GetOrderByIdAsync()`
- Do not use `.Result` or `.Wait()` on tasks — this risks deadlocks
- Pass `CancellationToken` through the call stack wherever IO is involved:

```csharp
public async Task<Order> GetOrderByIdAsync(Guid id, CancellationToken cancellationToken = default)
{
    return await _context.Orders.FindAsync(new object[] { id }, cancellationToken);
}
```

---

## Dependency Injection

- Register services in the appropriate lifetime: `Transient`, `Scoped`, or `Singleton`
- Inject interfaces, not concrete types
- Do not use service locator pattern (`IServiceProvider.GetService()`) except in infrastructure bootstrapping
- Use constructor injection; avoid property injection

---

## Error Handling

- Use custom exception types for domain errors (`OrderNotFoundException`, `PaymentFailedException`)
- Handle exceptions at the boundary — middleware for APIs, top-level handlers for workers
- Do not swallow exceptions silently (`catch (Exception) {}`)
- Log exceptions with enough context to diagnose the issue, but without sensitive data

```csharp
try
{
    await _paymentService.ProcessAsync(request, cancellationToken);
}
catch (PaymentFailedException ex)
{
    _logger.LogWarning(ex, "Payment failed for order {OrderId}", request.OrderId);
    throw;
}
```

---

## Null Handling

- Enable nullable reference types in all projects (`<Nullable>enable</Nullable>`)
- Use `?` to indicate nullable types explicitly; treat non-nullable types as guaranteed non-null
- Use the null-coalescing operator (`??`) and null-conditional operator (`?.`) for concise null checks
- Avoid `null` as a meaningful domain value — use `Option<T>` patterns or specific result types

---

## Entity Framework Core

- Use the repository pattern to abstract data access from application logic
- Never expose `DbContext` to controllers or application layer services
- Use `AsNoTracking()` for read-only queries
- Avoid lazy loading — use explicit `Include()` for required navigation properties
- Use migrations — never modify the database schema manually

---

## Unit Testing

- Follow the Arrange-Act-Assert (AAA) pattern
- Test method names: `MethodName_Scenario_ExpectedResult`
- Use `xUnit` as the test framework
- Use `Moq` or `NSubstitute` for mocking dependencies
- Aim for high coverage of domain logic and application use cases

```csharp
[Fact]
public async Task GetOrderByIdAsync_OrderExists_ReturnsOrder()
{
    // Arrange
    var orderId = Guid.NewGuid();
    _mockRepository.Setup(r => r.GetByIdAsync(orderId, default))
        .ReturnsAsync(new Order { Id = orderId });

    // Act
    var result = await _sut.GetOrderByIdAsync(orderId);

    // Assert
    Assert.NotNull(result);
    Assert.Equal(orderId, result.Id);
}
```
