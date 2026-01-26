# Project Instructions

> These instructions apply globally to all interactions with GitHub Copilot in this repository.

## Project Overview

AgenticAI - .NET 10 Template is a framework for building reliable AI workflows using Railway-Oriented Programming, Clean Architecture, and agentic primitives for context engineering.

## Tech Stack

- **Language**: C# 13.0
- **Runtime**: .NET 10
- **Pattern Library**: [CSharpFunctionalExtensions](https://github.com/vkhorikov/CSharpFunctionalExtensions/) (Railway-Oriented Programming / Result pattern)
- **Testing**: xUnit with Moq
- **ORM**: Entity Framework Core
- **Code Analysis**: 
  - SonarAnalyzer.CSharp
  - Microsoft.CodeAnalysis.NetAnalyzers
  - Meziantou.Analyzer
- **Documentation**: XML doc comments
- **Build Quality**: TreatWarningsAsErrors enabled

## Code Style & Conventions

### General Principles
- Write clean, readable, and maintainable code
- Follow the DRY (Don't Repeat Yourself) principle
- Prefer composition over inheritance
- Use meaningful variable and function names
- Embrace functional programming patterns with CSharpFunctionalExtensions
- Use async/await for all I/O operations
- Never ignore compiler warnings (TreatWarningsAsErrors = true)

### Formatting
- Use consistent indentation (4 spaces, enforced by .editorconfig)
- Keep lines under 120 characters when possible
- Use blank lines to separate logical blocks
- Run `dotnet format` before committing

### Naming Conventions
- **Files**: PascalCase matching class/interface name (e.g., `UserService.cs`)
- **Classes**: PascalCase (e.g., `UserService`, `CreateUserHandler`)
- **Interfaces**: PascalCase with I prefix (e.g., `IUserRepository`)
- **Methods**: PascalCase (e.g., `GetUserById`, `CreateUser`)
- **Properties**: PascalCase (e.g., `FirstName`, `IsActive`)
- **Private fields**: camelCase with _ prefix (e.g., `_userRepository`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)
- **Parameters**: camelCase (e.g., `userId`, `firstName`)
- **Local variables**: camelCase (e.g., `currentUser`)
- **Boolean properties/methods**: Start with Is/Has/Can (e.g., `IsActive`, `HasPermission`, `CanDelete`)

### C# Specific Conventions
- Use records for DTOs and immutable data (C# 9+)
- Use nullable reference types (`#nullable enable`)
- Use expression-bodied members for simple properties/methods
- Use pattern matching for null checks and type checks
- Use `var` for local variables when type is obvious from assignment
- Use explicit types for public APIs

## Railway-Oriented Programming (CSharpFunctionalExtensions)

### Result Pattern
Always use `Result<T>` or `Result` for operations that may fail:

```csharp
public Result<User> GetUser(string userId)
{
    if (string.IsNullOrEmpty(userId))
        return Result.Failure<User>("User ID is required");
    
    var user = _repository.Find(userId);
    return user is not null 
        ? Result.Success(user)
        : Result.Failure<User>("User not found");
}

// Chaining operations
public async Task<Result> CreateUserWithProfile(CreateUserRequest request)
{
    return await Result.Success(request)
        .Map(r => new User { Email = r.Email, Name = r.Name })
        .Bind(user => _repository.Add(user))
        .Bind(user => _profileService.CreateProfile(user.Id))
        .Tap(user => _logger.LogInformation($"User {user.Id} created"));
}
```

### Async Operations
```csharp
public async Task<Result<User>> GetUserAsync(string userId)
{
    // Use MapAsync and BindAsync for async operations
    return await Result.Success(userId)
        .MapAsync(id => _repository.FindAsync(id))
        .BindAsync(user => ValidateUserAsync(user));
}
```

## Project Structure

```
src/
├── [DomainName]/
│   ├── Entities/              # Domain entities
│   │   └── User.cs
│   ├── ValueObjects/          # Domain value objects
│   │   └── Email.cs
│   ├── Repositories/          # Data access abstractions
│   │   └── IUserRepository.cs
│   ├── Services/              # Business logic / Application services
│   │   └── UserService.cs
│   ├── Handlers/              # Command/Query handlers
│   │   ├── GetUserHandler.cs
│   │   └── CreateUserHandler.cs
│   └── DTOs/                  # Data transfer objects (records)
│       └── UserDto.cs

tests/
├── [DomainName].Tests/
│   ├── Unit/
│   │   └── Services/
│   │       └── UserServiceTests.cs
│   └── Integration/
│       └── Repositories/
│           └── UserRepositoryTests.cs
```

## Git Conventions

### Commit Messages
Use conventional commits format:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Code formatting (doesn't change behavior)
- `refactor:` - Code restructuring
- `test:` - Adding/updating tests
- `chore:` - Build, dependencies, tooling

Examples:
```
feat(user-service): implement railway-oriented GetUser method
fix(repository): handle null connection string gracefully
test(user-service): add edge cases for email validation
docs(readme): add CSharpFunctionalExtensions examples
```

### Branch Naming
- `feature/description` - New features
- `fix/description` - Bug fixes
- `docs/description` - Documentation updates
- `refactor/description` - Code refactoring

## Testing Requirements

- Write unit tests for all new services/handlers
- Maintain minimum 80% code coverage
- Include edge cases and error scenarios
- Use descriptive test names following AAA pattern (Arrange, Act, Assert)
- Mock external dependencies with Moq
- Test both success and failure paths

### Test Naming
```csharp
[Fact]
public void GetUser_WithValidUserId_ReturnsUser()
{
    // Arrange
    var userId = "123";
    
    // Act
    var result = _service.GetUser(userId);
    
    // Assert
    result.IsSuccess.Should().BeTrue();
}

[Fact]
public void GetUser_WithNullUserId_ReturnsFa failure()
{
    // Arrange
    var userId = null;
    
    // Act
    var result = _service.GetUser(userId);
    
    // Assert
    result.IsFailure.Should().BeTrue();
}
```

## Documentation

- Keep documentation minimal and focused
- Use XML doc comments for all public APIs
- Add comments only for complex or non-obvious logic
- Code should be self-documenting through clear naming and structure
- Use markdown for README and architectural docs

### XML Documentation Example
```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="userId">The unique identifier of the user to retrieve.</param>
/// <returns>
/// A <see cref="Result{User}"/> containing the user if found, 
/// or a failure result with an error message if not found or if input is invalid.
/// </returns>
/// <example>
/// <code>
/// var result = await userService.GetUserAsync("user-123");
/// if (result.IsSuccess)
/// {
///     Console.WriteLine($"User: {result.Value.Name}");
/// }
/// </code>
/// </example>
public async Task<Result<User>> GetUserAsync(string userId)
{
    // implementation
}
```

## Security Guidelines

- Never commit secrets or API keys (use appsettings.json + environment variables)
- Validate all user inputs using FluentValidation or similar
- Use Entity Framework Core's parameterized queries (automatic protection against SQL injection)
- Follow principle of least privilege
- Use `[Authorize]` and `[RequiredPermission]` attributes appropriately
- Never log sensitive data (passwords, tokens, PII)

## Error Handling

- Use Result<T> pattern from CSharpFunctionalExtensions
- Provide meaningful error messages
- Log errors with sufficient context
- Handle edge cases gracefully
- Don't throw exceptions for expected failures (use Result pattern)
- Only throw exceptions for truly exceptional cases

### Pattern Example
```csharp
public async Task<Result> SaveUserAsync(User user)
{
    try
    {
        if (user is null)
            return Result.Failure("User cannot be null");
        
        await _repository.SaveAsync(user);
        return Result.Success();
    }
    catch (OperationCanceledException)
    {
        return Result.Failure("Operation was cancelled");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to save user: {UserId}", user.Id);
        return Result.Failure("An unexpected error occurred while saving the user");
    }
}
```

## Code Analysis Requirements

All of these analyzers must pass with zero violations:
- **SonarAnalyzer.CSharp**: Code smells, security vulnerabilities, bug detection
- **Microsoft.CodeAnalysis.NetAnalyzers**: Best practices and API usage
- **Meziantou.Analyzer**: Modern C# features and performance suggestions

## Async/Await Guidelines

- Always use `async/await` for I/O operations (database, HTTP, file system)
- Return `Task` or `Task<T>` instead of `void` for async methods (except event handlers)
- Use `ConfigureAwait(false)` in library code
- Don't use `.Result` or `.Wait()` - it can cause deadlocks
- Use `async IEnumerable` for streaming operations


