# Project Architecture

> Document your project's architecture here. This file helps AI agents understand your system structure and technology patterns.

## Overview

This project is built on **.NET 10** using **Railway-Oriented Programming** patterns with **CSharpFunctionalExtensions** for reliable error handling and functional composition. The architecture emphasizes clean layers, functional purity, and type safety with nullable reference types.

## Technology Stack

- **Runtime**: .NET 10
- **Language**: C# 13.0
- **Pattern Library**: CSharpFunctionalExtensions (Railway-Oriented Programming / Result pattern)
- **ORM**: Entity Framework Core
- **Testing**: xUnit
- **Code Analysis**: 
  - SonarAnalyzer.CSharp
  - Microsoft.CodeAnalysis.NetAnalyzers
  - Meziantou.Analyzer
- **Build Quality**: TreatWarningsAsErrors enabled

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│         (ASP.NET Core Controllers / API Endpoints)          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         HTTP/REST API (RESTful endpoints)          │   │
│  │    Returns Result<T> wrapped in HTTP responses     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Application Service Layer                  │
│    (Command/Query Handlers using Railway-Oriented Pattern)  │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │  Command Handlers    │  │  Query Handlers      │        │
│  │  - Create User       │  │  - Get User          │        │
│  │  - Update User       │  │  - List Users        │        │
│  │  (Returns Result<T>) │  │  (Returns Result<T>) │        │
│  └──────────────────────┘  └──────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                     │
│            (Domain Services & Entities)                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • User Entity (Aggregate Root)                    │   │
│  │  • Value Objects (Email, PhoneNumber)             │   │
│  │  • Domain Services (UserService, etc.)            │   │
│  │  • Specification/Validation Logic                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Data Access Layer                       │
│            (Repositories & Entity Framework Core)           │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │  IUserRepository     │  │  IOrderRepository    │        │
│  │  (Abstraction)       │  │  (Abstraction)       │        │
│  └──────────────────────┘  └──────────────────────┘        │
│  ┌────────────────────────────────────────────────────┐    │
│  │  UserRepository, OrderRepository (EF Core impl)   │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                            │
│          (SQL Server / PostgreSQL / Database)               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │    Users     │  │    Orders    │  │   Profiles   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

## Layer Responsibilities

### Presentation Layer (Controllers)

- Handles HTTP requests/responses
- Maps DTOs to/from domain models
- Returns Result<T> wrapped in appropriate HTTP status codes
- Delegates business logic to handlers

```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<ApiResponse<UserDto>>> CreateAsync(
        [FromBody] CreateUserRequest request,
        [FromServices] ICommandHandler<CreateUserCommand, User> handler)
    {
        var result = await handler.Handle(new CreateUserCommand(request));
        return result.IsSuccess 
            ? Ok(ApiResponse.Success(new UserDto(result.Value)))
            : BadRequest(ApiResponse.Error(result.Error));
    }
}
```

### Application Service Layer (Handlers)

- Implements Commands and Queries (CQRS pattern)
- Orchestrates domain logic with repositories
- Returns Result<T> for all operations
- Uses Railway-Oriented Programming for error handling

```csharp
public class CreateUserHandler : ICommandHandler<CreateUserCommand, User>
{
    public async Task<Result<User>> Handle(CreateUserCommand command)
    {
        return await Result.Success(command)
            .MapAsync(cmd => User.Create(cmd.Name, cmd.Email))
            .BindAsync(user => _repository.AddAsync(user))
            .TapAsync(user => _eventBus.PublishAsync(new UserCreatedEvent(user.Id)));
    }
}
```

### Business Logic Layer (Services & Entities)

- Defines domain entities and value objects
- Encapsulates business rules
- No dependencies on infrastructure
- Uses immutability and functional patterns

```csharp
public class User
{
    public string Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; }
    public bool IsActive { get; private set; }
    
    public static Result<User> Create(string name, string email)
    {
        if (string.IsNullOrEmpty(name))
            return Result.Failure<User>("Name is required");
        
        var emailResult = Email.Create(email);
        return emailResult
            .Map(validEmail => new User 
            { 
                Id = Guid.NewGuid().ToString(),
                Name = name,
                Email = validEmail,
                IsActive = true
            });
    }
}
```

### Data Access Layer (Repositories)

- Abstracts database operations
- Implements repositories for each aggregate
- Returns Result<T> for all operations
- Uses Entity Framework Core for queries

```csharp
public interface IUserRepository
{
    Task<Result<User>> GetByIdAsync(string userId);
    Task<Result<User>> AddAsync(User user);
    Task<Result> UpdateAsync(User user);
    Task<Result> DeleteAsync(string userId);
}

public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Result<User>> GetByIdAsync(string userId)
    {
        var user = await _context.Users.FirstOrDefaultAsync(u => u.Id == userId);
        return user is not null
            ? Result.Success(user)
            : Result.Failure<User>($"User '{userId}' not found");
    }
}
```

## Railway-Oriented Programming Pattern

The project uses **Railway-Oriented Programming** (ROP) with `CSharpFunctionalExtensions` to handle errors functionally:

```
Input
  │
  ▼
┌─────────────────┐
│  Happy Path     │─ Success(value) ─►┐
└─────────────────┘                    │
                                       │
  Validation Error ──► Failure(error) ─┤
                                       │
Database Error ────► Failure(error) ────┤
                                       │
                                       ▼
                                    Output
```

**Map**: Transform success value
```csharp
Result.Success(5)
    .Map(x => x * 2)  // Success(10)
```

**Bind**: Chain operations that return Result
```csharp
Result.Success(userId)
    .Bind(id => GetUser(id))           // Result<User>
    .Bind(user => UpdateUser(user))    // Result<User>
```

**Tap**: Execute side effects (logging, events)
```csharp
Result.Success(user)
    .Tap(u => _logger.Log($"User {u.Id} created"))
    .Tap(u => _eventBus.Publish(new UserCreatedEvent(u.Id)))
```

**MapAsync/BindAsync**: Async versions
```csharp
Result.Success(userId)
    .MapAsync(id => FetchFromApiAsync(id))
    .BindAsync(user => _repository.SaveAsync(user))
```

## Project Structure

```
src/
├── Core/                           # Core/shared infrastructure
│   ├── Result.cs                  # Result<T> helper (might use library)
│   └── Constants.cs
│
├── Domain/                         # Domain-agnostic business logic
│   ├── Entities/
│   │   ├── User.cs
│   │   └── Order.cs
│   ├── ValueObjects/
│   │   ├── Email.cs
│   │   ├── Money.cs
│   │   └── PhoneNumber.cs
│   ├── Repositories/
│   │   ├── IUserRepository.cs
│   │   └── IOrderRepository.cs
│   └── Events/
│       ├── UserCreatedEvent.cs
│       └── OrderPlacedEvent.cs
│
├── Application/                    # Use cases & orchestration
│   ├── Commands/
│   │   ├── CreateUserCommand.cs
│   │   └── CreateUserHandler.cs
│   ├── Queries/
│   │   ├── GetUserQuery.cs
│   │   └── GetUserHandler.cs
│   ├── DTOs/
│   │   ├── UserDto.cs
│   │   ├── CreateUserRequest.cs
│   │   └── UpdateUserRequest.cs
│   └── Services/
│       └── UserService.cs
│
├── Infrastructure/                 # Database, external services
│   ├── Persistence/
│   │   ├── ApplicationDbContext.cs
│   │   ├── UserRepository.cs
│   │   └── Migrations/
│   ├── Events/
│   │   └── EventBus.cs
│   └── Configuration/
│       └── DependencyInjection.cs
│
└── Presentation/                   # ASP.NET Core API
    └── Controllers/
        ├── UsersController.cs
        └── OrdersController.cs

tests/
├── Domain.Tests/                  # Domain logic tests
│   ├── Entities/
│   │   └── UserTests.cs
│   └── ValueObjects/
│       └── EmailTests.cs
│
├── Application.Tests/             # Handler/service tests
│   ├── Commands/
│   │   └── CreateUserHandlerTests.cs
│   └── Queries/
│       └── GetUserHandlerTests.cs
│
└── Integration.Tests/             # Repository/API tests
    ├── Repositories/
    │   └── UserRepositoryTests.cs
    └── API/
        └── UsersControllerTests.cs
```

## Key Design Decisions

### Decision 1: Railway-Oriented Programming with CSharpFunctionalExtensions

**Context**: Need reliable error handling without exceptions or null coalescing chains

**Decision**: Use `Result<T>` pattern throughout the application

**Rationale**:
- Explicit error handling (no silent failures)
- Composable operations via Map/Bind/Tap
- Type-safe error propagation
- Reduced null-reference exceptions

**Trade-offs**:
- Learning curve for new developers
- More verbose than exceptions
- Requires consistent adoption

### Decision 2: Nullable Reference Types Enabled

**Context**: C# null-reference exceptions are common runtime errors

**Decision**: Enable `#nullable enable` globally with `TreatWarningsAsErrors`

**Rationale**:
- Compile-time null safety
- Reduces null-reference exceptions at runtime
- Improved code documentation (string vs string?)

**Trade-offs**:
- Additional compiler warnings to address
- Initial setup complexity
- Some third-party libraries may not support

### Decision 3: Strict Code Analysis

**Context**: Need consistent code quality across the project

**Decision**: Enable SonarAnalyzer, Microsoft.CodeAnalysis.NetAnalyzers, Meziantou.Analyzer with TreatWarningsAsErrors

**Rationale**:
- Early detection of bugs and vulnerabilities
- Enforced best practices
- Improved maintainability

**Trade-offs**:
- Strict compilation requirements
- May restrict some legitimate patterns
- Requires understanding of analyzer rules

## External Dependencies & APIs (Context7)

This project uses **Context7** MCP for real-time documentation lookups:

### CSharpFunctionalExtensions
- **Usage**: Result<T> pattern for functional error handling
- **Documentation**: Auto-fetched via Context7
- **Examples**: Railway-Oriented Programming chains

### Entity Framework Core
- **Usage**: ORM for database access
- **Documentation**: Auto-fetched for query patterns
- **Examples**: Async queries, lazy loading, change tracking

### xUnit
- **Usage**: Unit and integration testing framework
- **Documentation**: Auto-fetched for testing patterns
- **Examples**: Fact/Theory attributes, assertions

### SonarAnalyzer.CSharp
- **Usage**: Code quality analysis
- **Documentation**: Auto-fetched for rule violations
- **Examples**: Security checks, code smells

## Quality Gates

All code must pass these gates before merging:

1. **Build**: `dotnet build` - Zero warnings/errors
2. **Static Analysis**: All analyzers pass (SonarAnalyzer, NetAnalyzers, Meziantou)
3. **Tests**: >= 80% code coverage, all tests pass
4. **Formatting**: `dotnet format` produces no changes
5. **Documentation**: All public APIs documented
6. **Microsoft Best Practices**: Compliant with ASP.NET Core best practices (see [aspnet-core-best-practices.md](.github/instructions/aspnet-core-best-practices.md))

## Performance & Reliability

This architecture follows Microsoft ASP.NET Core best practices for:

- **Zero Blocking Calls**: 100% async/await, no Task.Result/Wait()
- **Thread Pool Efficiency**: No thread starvation, no wasteful Task.Run()
- **Memory Safety**: HttpContext properly scoped, no storage in fields
- **Request/Response Safety**: Async APIs for body handling, no large buffering
- **Caching Strategy**: Aggressive caching with proper invalidation
- **Database Optimization**: Paginated results, no N+1 queries, AsNoTracking for reads
- **Connection Pooling**: HttpClientFactory for external APIs
- **Exception Handling**: Result<T> pattern minimizes exceptions

For detailed best practices guidance, see [Microsoft ASP.NET Core Best Practices](.github/instructions/aspnet-core-best-practices.md)


|------------|---------|---------------|
| Service A | Authentication | [Link] |
| Service B | Payments | [Link] |

## Development Setup

Refer to [README.md](../README.md) for setup instructions.

## Related Documentation

- [Conventions](./conventions.md)
- [API Documentation](./api.md)
