# Coding Conventions

> This document defines the coding standards for this project. AI agents and developers should follow these conventions.

## General Principles

1. **Clarity over cleverness**: Write code that is easy to understand
2. **Consistency**: Follow established patterns in the codebase
3. **Simplicity**: Prefer simple solutions over complex ones
4. **Documentation**: Document the "why", not the "what"
5. **Functional-First**: Embrace immutability and composition patterns

## Code Style

### Formatting

- **Indentation**: 4 spaces (enforced by .editorconfig)
- **Line length**: Maximum 120 characters
- **Braces**: Allman style (opening brace on new line)
- **Trailing commas**: Yes, in multiline structures
- **Auto-formatting**: Use `dotnet format` before committing

Example:
```csharp
public class UserService
{
    public async Task<Result<User>> GetUserAsync(string userId)
    {
        var user = await _repository.GetByIdAsync(userId);
        return user;
    }
}
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | PascalCase (match class name) | `UserService.cs`, `IUserRepository.cs` |
| Classes | PascalCase | `UserService` |
| Interfaces | PascalCase with I prefix | `IUserRepository` |
| Methods | PascalCase | `GetUserById`, `CreateUserAsync` |
| Properties | PascalCase | `FirstName`, `IsActive`, `CreatedAt` |
| Private fields | camelCase with _ prefix | `_userRepository`, `_logger` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |
| Local variables | camelCase | `userId`, `isValid`, `tempList` |
| Parameters | camelCase | `userId`, `firstName`, `options` |
| Boolean methods/properties | Is/Has/Can prefix | `IsActive`, `HasPermission`, `CanDelete` |
| Async methods | Async suffix | `GetUserAsync`, `CreateUserAsync` |
| Records (DTOs) | PascalCase | `UserDto`, `CreateUserRequest` |
| Enums | PascalCase | `UserStatus`, `OrderState` |

### File Organization

```
src/
├── [DomainName]/
│   ├── Entities/              # Domain entities
│   │   └── User.cs
│   ├── ValueObjects/          # Value objects for domain concepts
│   │   └── Email.cs
│   │   └── PhoneNumber.cs
│   ├── Repositories/          # Data access abstractions
│   │   └── IUserRepository.cs
│   ├── Services/              # Business logic / Application services
│   │   └── UserService.cs
│   ├── Handlers/              # Command/Query handlers (CQRS pattern)
│   │   ├── GetUserHandler.cs
│   │   └── CreateUserHandler.cs
│   ├── DTOs/                  # Data transfer objects
│   │   ├── UserDto.cs
│   │   ├── CreateUserRequest.cs
│   │   └── UpdateUserRequest.cs
│   └── Exceptions/            # Domain-specific exceptions
│       └── UserNotFoundException.cs

tests/
├── [DomainName].Tests/
│   ├── Unit/
│   │   ├── Services/
│   │   │   └── UserServiceTests.cs
│   │   └── Handlers/
│   │       └── GetUserHandlerTests.cs
│   └── Integration/
│       └── Repositories/
│           └── UserRepositoryTests.cs
```

## C# Specific Conventions

### Nullable Reference Types

Always enable nullable reference types and use them properly:

```csharp
#nullable enable

public class User
{
    // Non-nullable property - must have a value
    public string Id { get; set; }
    
    // Nullable property - can be null
    public string? MiddleName { get; set; }
    
    // Required property (C# 11+)
    public required string Email { get; set; }
}

// Null-coalescing and null-conditional operators
var name = user?.MiddleName ?? user.FirstName;
```

### Using Declarations

Use `using` declarations for automatic resource disposal:

```csharp
// ✅ DO: Using declaration (C# 8+)
using var connection = new SqlConnection(connectionString);
// Connection automatically disposed at end of scope

// ❌ DON'T: Old-style using statement
using (var connection = new SqlConnection(connectionString))
{
    // connection disposed here
}
```

### Record Types

Use records for immutable data and DTOs:

```csharp
// Value object pattern
public record Email(string Value)
{
    public static Result<Email> Create(string value)
    {
        if (string.IsNullOrEmpty(value))
            return Result.Failure<Email>("Email cannot be empty");
        
        if (!value.Contains("@"))
            return Result.Failure<Email>("Invalid email format");
        
        return Result.Success(new Email(value));
    }
}

// DTO
public record UserDto(string Id, string Name, string Email);

// Request object
public record CreateUserRequest(string Name, string Email, string Password);
```

### Expression-Bodied Members

Use for simple properties and methods:

```csharp
// ✅ DO: Expression-bodied for simple implementations
public class User
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
    public string FullName => $"{FirstName} {LastName}";
    public bool IsActive => !IsDeleted && IsVerified;
}

// ❌ DON'T: For complex logic
public string GetDisplayName() => 
    string.IsNullOrEmpty(MiddleName) 
        ? $"{FirstName} {LastName}" 
        : $"{FirstName} {MiddleName} {LastName}";  // Too complex!
```

### Pattern Matching

Use modern pattern matching for null checks and type checks:

```csharp
// ✅ DO: Use pattern matching
public decimal CalculateDiscount(User user) => user switch
{
    null => throw new ArgumentNullException(nameof(user)),
    { IsVip: true } => 0.20m,
    { IsActive: true, Years: > 5 } => 0.15m,
    { IsActive: true } => 0.10m,
    _ => 0m
};

// ✅ DO: Null-coalescing pattern
if (order?.Items?.FirstOrDefault() is not null)
{
    // Use order items
}
```

### Var Usage

Use `var` for local variables when the type is obvious from the assignment:

```csharp
// ✅ DO: Clear types
var userId = "123";  // var is fine - string is obvious
var users = await _repository.GetAllAsync();  // var is fine - method name indicates users
var count = users.Count();  // var is fine - Count() returns int

// ❌ DON'T: Unclear types
var x = GetData();  // What is x? Unclear!
var result = _service.Process();  // What type? Unclear!

// ✅ DO: Use explicit type for public APIs
public Result<User> GetUser(string userId)  // Not: public var GetUser...
{
    return _repository.GetById(userId);
}
```

## Methods and Functions

### Guidelines

- Keep methods small (< 30 lines ideally)
- Single Responsibility Principle
- Maximum 3-4 parameters (use options objects for more)
- Always mark methods as `async` if they perform I/O
- Use meaningful names that describe behavior

### Documentation

```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="userId">The unique identifier of the user to retrieve.</param>
/// <returns>
/// A <see cref="Result{User}"/> containing the user if found,
/// or a failure result with an error message if not found or if input is invalid.
/// </returns>
/// <exception cref="ArgumentNullException">Thrown when userId is null.</exception>
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
    if (userId == null)
        throw new ArgumentNullException(nameof(userId));
    
    return await _repository.GetByIdAsync(userId);
}
```

### Error Handling

```csharp
// ✅ DO: Return Result<T> for expected failures
public Result<User> ValidateEmail(string email)
{
    if (string.IsNullOrEmpty(email))
        return Result.Failure<User>("Email is required");
    
    if (!email.Contains("@"))
        return Result.Failure<User>("Invalid email format");
    
    return Result.Success(new User { Email = email });
}

// ✅ DO: Use try-catch for unexpected exceptions
public async Task<Result<User>> CreateUserAsync(User user)
{
    try
    {
        await _repository.AddAsync(user);
        return Result.Success(user);
    }
    catch (DbUpdateException ex)
    {
        _logger.LogError(ex, "Database error creating user");
        return Result.Failure<User>("Failed to create user");
    }
}

// ❌ DON'T: Swallow errors silently
try
{
    await _repository.SaveAsync(user);
}
catch
{
    // Bad: error is ignored!
}
```

## Testing

### Naming Convention

Use the format: `MethodName_Scenario_ExpectedResult`

```csharp
[Trait("Category", "Unit")]
public class UserServiceTests
{
    [Fact]
    public void GetUser_WithValidUserId_ReturnsUser()
    {
        // Arrange
        var userId = "123";
        
        // Act
        var result = _service.GetUser(userId);
        
        // Assert
        result.Should().NotBeNull();
    }
    
    [Fact]
    public void GetUser_WithNullUserId_ReturnsFailure()
    {
        // Arrange
        var userId = (string)null;
        
        // Act
        var result = _service.GetUser(userId);
        
        // Assert
        result.IsFailure.Should().BeTrue();
    }
    
    [Theory]
    [InlineData("")]
    [InlineData(null)]
    [InlineData("   ")]
    public void GetUser_WithInvalidUserId_ReturnsFailure(string userId)
    {
        // Act
        var result = _service.GetUser(userId);
        
        // Assert
        result.IsFailure.Should().BeTrue();
    }
}
```

### Structure: AAA Pattern

```csharp
[Fact]
public async Task CreateUserAsync_WithValidRequest_CreatesAndReturnsUser()
{
    // Arrange
    var request = new CreateUserRequest { Name = "John Doe", Email = "john@example.com" };
    var mockRepository = new Mock<IUserRepository>();
    mockRepository.Setup(r => r.AddAsync(It.IsAny<User>()))
        .ReturnsAsync(Result.Success(new User { Name = "John Doe" }));
    var service = new UserService(mockRepository.Object);
    
    // Act
    var result = await service.CreateUserAsync(request);
    
    // Assert
    result.IsSuccess.Should().BeTrue();
    result.Value.Name.Should().Be("John Doe");
    mockRepository.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Once);
}
```

## Git Conventions

### Commit Messages

Format: `<type>(<scope>): <description>`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting (no logic change)
- `refactor`: Code restructuring
- `test`: Adding/updating tests
- `chore`: Maintenance/tooling
- `perf`: Performance improvement

Examples:
```
feat(user-service): implement railway-oriented GetUser method
fix(repository): handle null connection string gracefully
test(user-service): add edge cases for email validation
docs(readme): add CSharpFunctionalExtensions examples
perf(query): optimize N+1 query in GetUsersWithProfiles
```

### Branch Naming
- `feature/user-authentication`
- `fix/null-reference-exception`
- `docs/api-documentation`
- `refactor/simplify-service-logic`



### Branch Naming

- `feature/short-description`
- `fix/issue-number-description`
- `docs/what-is-documented`
- `refactor/what-is-refactored`

## Code Review Checklist

Before submitting for review:

- [ ] Code follows project conventions
- [ ] Tests are written and passing
- [ ] No lint or type errors
- [ ] Documentation is updated
- [ ] Commit messages are clear
- [ ] PR description explains the changes

## Resources

- [Effective TypeScript](https://effectivetypescript.com/)
- [Clean Code principles](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
