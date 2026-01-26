---
applyTo: "**/*.cs"
description: "C#/.NET Backend Development Guidelines"
---

# C#/.NET Backend Development Guidelines

This guide applies to all C# backend code in this project built on .NET 10 with Railway-Oriented Programming patterns.

## Core Principles

1. **Railway-Oriented Programming**: Use `Result<T>` pattern from CSharpFunctionalExtensions for all operations that may fail
2. **Async-First**: Always use `async/await` for I/O operations
3. **Type Safety**: Enable nullable reference types (`#nullable enable`)
4. **Functional Style**: Prefer immutability, composition, and higher-order functions
5. **Code Quality**: TreatWarningsAsErrors = true - all warnings must be addressed

## Railway-Oriented Programming with CSharpFunctionalExtensions

### Result Pattern Fundamentals

```csharp
using CSharpFunctionalExtensions;

// Operations return Result<T> indicating success or failure
public Result<User> GetUser(string userId)
{
    if (string.IsNullOrEmpty(userId))
        return Result.Failure<User>("User ID cannot be empty");
    
    var user = _repository.Find(userId);
    return user is not null
        ? Result.Success(user)
        : Result.Failure<User>($"User '{userId}' not found");
}

// Void operations return Result (without value)
public async Task<Result> UpdateUserAsync(string userId, string newName)
{
    var userResult = await GetUserAsync(userId);
    
    return await userResult
        .Map(user => user.UpdateName(newName))
        .BindAsync(user => _repository.SaveAsync(user));
}
```

### Chaining Operations

```csharp
// Using Map for transformations
public Result<UserDto> GetUserAsDto(string userId)
{
    return GetUser(userId)
        .Map(user => new UserDto 
        { 
            Id = user.Id, 
            Name = user.Name, 
            Email = user.Email.Value 
        });
}

// Using Bind for operations that return Result
public async Task<Result<UserWithProfile>> GetUserWithProfileAsync(string userId)
{
    return await GetUserAsync(userId)
        .BindAsync(user => _profileService.GetProfileAsync(user.Id)
            .MapAsync(profile => new UserWithProfile(user, profile)));
}

// Using Tap for side effects (logging, tracking, etc.)
public async Task<Result<User>> CreateUserAsync(CreateUserRequest request)
{
    return await Result.Success(request)
        .MapAsync(req => new User { Email = req.Email, Name = req.Name })
        .BindAsync(user => _repository.AddAsync(user))
        .TapAsync(user => _logger.LogInformation("User created: {UserId}", user.Id))
        .TapAsync(user => _eventBus.PublishAsync(new UserCreatedEvent(user.Id)));
}
```

## API Design

### RESTful Principles
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes (200, 201, 400, 404, 409, 500)
- Use plural nouns for resource endpoints: `/api/v1/users`
- Version your APIs: `/api/v1/`, `/api/v2/`
- Implement pagination for list endpoints with `limit`, `offset`

### Response Format

```csharp
// Success response
{
  "success": true,
  "data": { /* resource */ },
  "timestamp": "2024-01-26T10:30:00Z"
}

// Error response
{
  "success": false,
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Email is required",
      "field": "email"
    }
  ],
  "timestamp": "2024-01-26T10:30:00Z"
}

// Paginated response
{
  "success": true,
  "data": [ /* items */ ],
  "pagination": {
    "limit": 10,
    "offset": 0,
    "total": 100,
    "hasMore": true
  },
  "timestamp": "2024-01-26T10:30:00Z"
}
```

### Controller Pattern

```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    [HttpGet("{userId}")]
    public async Task<ActionResult<ApiResponse<UserDto>>> GetUserAsync(string userId)
    {
        var result = await _userService.GetUserAsync(userId);
        
        // Result pattern maps to HTTP responses
        return result
            .Map(user => Ok(ApiResponse.Success(user)))
            .MapError(error => NotFound(ApiResponse.Error(error)))
            .Value;
    }
    
    [HttpPost]
    public async Task<ActionResult<ApiResponse<UserDto>>> CreateUserAsync([FromBody] CreateUserRequest request)
    {
        var result = await _userService.CreateUserAsync(request);
        
        return result
            .Map(user => CreatedAtAction(nameof(GetUserAsync), new { userId = user.Id }, 
                ApiResponse.Success(new UserDto(user))))
            .MapError(error => BadRequest(ApiResponse.Error(error)))
            .Value;
    }
}
```

## Database Operations

### Entity Framework Core Best Practices

```csharp
// Repository pattern with Result
public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Result<User>> GetByIdAsync(string userId)
    {
        var user = await _context.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == userId);
        
        return user is not null
            ? Result.Success(user)
            : Result.Failure<User>($"User '{userId}' not found");
    }
    
    public async Task<Result<User>> AddAsync(User user)
    {
        try
        {
            _context.Users.Add(user);
            await _context.SaveChangesAsync();
            return Result.Success(user);
        }
        catch (DbUpdateException ex) when (ex.InnerException is UniqueConstraintException)
        {
            return Result.Failure<User>("A user with this email already exists");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error adding user");
            return Result.Failure<User>("An unexpected error occurred while adding the user");
        }
    }
    
    public async Task<Result> DeleteAsync(string userId)
    {
        var user = await _context.Users.FindAsync(userId);
        if (user is null)
            return Result.Failure($"User '{userId}' not found");
        
        _context.Users.Remove(user);
        await _context.SaveChangesAsync();
        return Result.Success();
    }
}

// Query optimization
public async Task<Result<IEnumerable<UserDto>>> GetActiveUsersAsync(int limit = 10, int offset = 0)
{
    var users = await _context.Users
        .AsNoTracking()
        .Where(u => u.IsActive)
        .OrderByDescending(u => u.CreatedAt)
        .Skip(offset)
        .Take(limit)
        .Select(u => new UserDto 
        { 
            Id = u.Id, 
            Name = u.Name, 
            Email = u.Email.Value 
        })
        .ToListAsync();
    
    return Result.Success((IEnumerable<UserDto>)users);
}
```

### Handling N+1 Queries

```csharp
// ❌ DON'T: N+1 queries
public async Task<List<UserWithProfilesDto>> GetUsersWithProfilesAsync()
{
    var users = await _context.Users.ToListAsync();
    var result = new List<UserWithProfilesDto>();
    
    foreach (var user in users)
    {
        var profiles = await _context.Profiles
            .Where(p => p.UserId == user.Id)
            .ToListAsync(); // 1 + N queries!
        
        result.Add(new UserWithProfilesDto(user, profiles));
    }
    
    return result;
}

// ✅ DO: Single query with Include/Join
public async Task<Result<IEnumerable<UserWithProfilesDto>>> GetUsersWithProfilesAsync()
{
    var result = await _context.Users
        .AsNoTracking()
        .Include(u => u.Profiles)
        .Select(u => new UserWithProfilesDto(
            u.Id,
            u.Name,
            u.Profiles.Select(p => new ProfileDto(p.Id, p.Name)).ToList()
        ))
        .ToListAsync();
    
    return Result.Success((IEnumerable<UserWithProfilesDto>)result);
}
```

## Validation

### Input Validation

```csharp
// Use FluentValidation for complex rules
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Email format is invalid");
        
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .MaximumLength(100).WithMessage("Name cannot exceed 100 characters");
    }
}

// Validate before business logic
public async Task<Result> CreateUserAsync(CreateUserRequest request)
{
    var validationResult = await _validator.ValidateAsync(request);
    if (!validationResult.IsValid)
    {
        var errors = validationResult.Errors
            .Select(e => e.ErrorMessage)
            .Aggregate((a, b) => $"{a}; {b}");
        return Result.Failure(errors);
    }
    
    // Proceed with business logic
    return await _repository.AddAsync(new User { Email = request.Email });
}
```

## Error Handling

### Custom Error Types

```csharp
public abstract record DomainError(string Message);

public record ValidationError(string Message, string? Field = null) 
    : DomainError(Message);

public record NotFoundError(string Message) 
    : DomainError(Message);

public record ConflictError(string Message) 
    : DomainError(Message);

public record UnauthorizedError(string Message) 
    : DomainError(Message);

// Using custom errors
public Result<User> ValidateEmail(string email)
{
    if (string.IsNullOrEmpty(email))
        return Result.Failure<User>(new ValidationError("Email is required", nameof(email)));
    
    if (!email.Contains("@"))
        return Result.Failure<User>(new ValidationError("Invalid email format", nameof(email)));
    
    return Result.Success(/* user */);
}
```

### Exception Handling

```csharp
public async Task<Result<User>> GetUserAsync(string userId)
{
    try
    {
        var user = await _repository.GetByIdAsync(userId);
        return user is not null
            ? Result.Success(user)
            : Result.Failure<User>(new NotFoundError($"User '{userId}' not found"));
    }
    catch (OperationCanceledException)
    {
        _logger.LogInformation("Request cancelled for user {UserId}", userId);
        return Result.Failure<User>(new DomainError("Request was cancelled"));
    }
    catch (TimeoutException)
    {
        _logger.LogError("Timeout while fetching user {UserId}", userId);
        return Result.Failure<User>(new DomainError("Request timed out"));
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error fetching user {UserId}", userId);
        return Result.Failure<User>(new DomainError("An unexpected error occurred"));
    }
}
```

## Async/Await Best Practices

```csharp
// ✅ DO: Use async/await for all I/O operations
public async Task<Result<User>> GetUserAsync(string userId)
{
    return await _repository.GetByIdAsync(userId);
}

// ❌ DON'T: Use .Result or .Wait() - causes deadlocks
public Result<User> GetUser(string userId)
{
    return _repository.GetByIdAsync(userId).Result; // DEADLOCK RISK!
}

// ✅ DO: Use ConfigureAwait(false) in library code
public async Task<Result<User>> GetUserAsync(string userId)
{
    var user = await _repository.GetByIdAsync(userId).ConfigureAwait(false);
    return user;
}

// ✅ DO: Return Task/Task<T>, not void
public async Task UpdateUserAsync(User user)
{
    await _repository.SaveAsync(user);
}

// ❌ DON'T: Return void from async methods (except event handlers)
public async void UpdateUser(User user) // Exception will crash app!
{
    await _repository.SaveAsync(user);
}

// ✅ DO: Use async IEnumerable for streaming
public async IAsyncEnumerable<User> GetAllUsersAsync()
{
    var page = 0;
    const int pageSize = 100;
    
    while (true)
    {
        var users = await _repository.GetPageAsync(page++, pageSize);
        if (users.Count == 0) yield break;
        
        foreach (var user in users)
            yield return user;
    }
}
```

## Performance Optimization

```csharp
// Use AsNoTracking for read-only queries
var users = await _context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync();

// Use Select to project only needed fields
var userEmails = await _context.Users
    .AsNoTracking()
    .Select(u => new { u.Id, u.Email })
    .ToListAsync();

// Use efficient pagination
public async Task<Result<PaginatedResult<UserDto>>> GetUsersAsync(int page, int pageSize)
{
    var total = await _context.Users.CountAsync();
    var users = await _context.Users
        .AsNoTracking()
        .OrderBy(u => u.Id)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(u => new UserDto(u.Id, u.Name))
        .ToListAsync();
    
    return Result.Success(new PaginatedResult<UserDto>(users, total, page, pageSize));
}

// Use batch operations for multiple updates
public async Task<Result> UpdateMultipleUsersAsync(IEnumerable<User> users)
{
    _context.Users.UpdateRange(users);
    await _context.SaveChangesAsync();
    return Result.Success();
}
```

## Logging

```csharp
private readonly ILogger<UserService> _logger;

// ✅ DO: Use structured logging with parameters
_logger.LogInformation("User created: {UserId} {Email}", user.Id, user.Email);

// ✅ DO: Use appropriate log levels
_logger.LogDebug("Fetching user: {UserId}", userId);
_logger.LogInformation("User updated successfully");
_logger.LogWarning("User not found: {UserId}", userId);
_logger.LogError(ex, "Error creating user: {Email}", email);

// ❌ DON'T: Log sensitive data
_logger.LogInformation("User password: {Password}", password); // NEVER!

// ✅ DO: Use structured exceptions in logging
try
{
    await _repository.SaveAsync(user);
}
catch (Exception ex)
{
    _logger.LogError(ex, "Failed to save user {UserId}", user.Id);
    return Result.Failure("Failed to save user");
}
```

## Testing

```csharp
[Trait("Category", "Unit")]
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _repositoryMock;
    private readonly UserService _service;
    
    public UserServiceTests()
    {
        _repositoryMock = new Mock<IUserRepository>();
        _service = new UserService(_repositoryMock.Object);
    }
    
    [Fact]
    public async Task GetUserAsync_WithValidUserId_ReturnsUser()
    {
        // Arrange
        var userId = "123";
        var expectedUser = new User { Id = userId, Name = "John Doe" };
        _repositoryMock
            .Setup(r => r.GetByIdAsync(userId))
            .ReturnsAsync(Result.Success(expectedUser));
        
        // Act
        var result = await _service.GetUserAsync(userId);
        
        // Assert
        result.IsSuccess.Should().BeTrue();
        result.Value.Id.Should().Be(userId);
        result.Value.Name.Should().Be("John Doe");
        _repositoryMock.Verify(r => r.GetByIdAsync(userId), Times.Once);
    }
    
    [Fact]
    public async Task GetUserAsync_WithInvalidUserId_ReturnsFailure()
    {
        // Arrange
        var userId = "invalid";
        _repositoryMock
            .Setup(r => r.GetByIdAsync(userId))
            .ReturnsAsync(Result.Failure<User>("User not found"));
        
        // Act
        var result = await _service.GetUserAsync(userId);
        
        // Assert
        result.IsFailure.Should().BeTrue();
        result.Error.Should().Contain("not found");
    }
}
```


