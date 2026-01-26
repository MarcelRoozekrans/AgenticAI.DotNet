---
applyTo: "Code Review Process"
description: "Architect/Reviewer Checklist for ASP.NET Core Best Practices"
---

# Architect & Reviewer Checklist

> This checklist ensures code reviews validate both Railway-Oriented Programming patterns AND Microsoft ASP.NET Core best practices.
> Use this during code review/architectural review phases.

## Pre-Review: Automated Checks

Before manual review, verify automated gates pass:

```bash
# Format check
dotnet format --verify-no-changes

# Build with strict warnings
dotnet build /p:TreatWarningsAsErrors=true

# Run all tests with coverage
dotnet test /p:CollectCoverage=true

# Check for vulnerable dependencies
dotnet list package --vulnerable
```

**All must show GREEN ✅ before proceeding to manual review.**

---

## Phase 1: Async/Await Compliance (CRITICAL)

> Thread pool starvation is the #1 performance issue. This is non-negotiable.

### Checklist

- [ ] **No blocking calls in hot paths**
  - [ ] No `.Result` or `.Wait()` anywhere
  - [ ] No `Task.Run()` followed by `await`
  - [ ] No synchronous wrappers for async APIs
  - Command: `grep -r "\.Result\|\.Wait()\|Task\.Run" src/`

- [ ] **All I/O operations are async**
  - [ ] Database calls use `*Async` methods
  - [ ] HTTP calls use async HttpClient (via IHttpClientFactory)
  - [ ] File I/O uses `*Async` methods
  - [ ] Stream operations use `*Async` methods (ReadAsync, ReadToEndAsync, ReadFormAsync, DeserializeAsync)
  - Look for: `.Read(`, `.Write(`, `.Form` (sync versions)

- [ ] **Controller actions return Task<ActionResult>**
  - [ ] No `IActionResult` without `async Task<>`
  - [ ] No `ActionResult<T>` from sync methods
  - Example: `public async Task<ActionResult<UserDto>>`, not `public ActionResult<UserDto>`

- [ ] **No async void methods**
  - [ ] Exception: Only event handlers allowed
  - [ ] Command: `grep -r "async void" src/ --exclude="*Event*.cs"`
  - Event handlers are acceptable: `private async void OnPropertyChanged`

- [ ] **CancellationToken support**
  - [ ] All async methods accept `CancellationToken cancellationToken = default`
  - [ ] Tokens passed to underlying async calls
  - [ ] Database context uses tokens
  - [ ] HttpClient uses tokens

---

## Phase 2: HttpContext Safety (CRITICAL)

> HttpContext is request-scoped and not thread-safe. Incorrect usage causes crashes.

### Checklist

- [ ] **HttpContext not stored in fields**
  - [ ] No `private HttpContext _context` fields
  - [ ] No `private IHttpContextAccessor _accessor` with access to `.HttpContext` stored
  - [ ] Pattern: Store `IHttpContextAccessor`, access `.HttpContext` in methods

- [ ] **HttpContext not accessed from parallel operations**
  - [ ] Before `Task.WhenAll()`, copy needed values
  - [ ] Look for: Any access to `HttpContext` inside tasks passed to `Task.WhenAll`
  - [ ] Pattern:
    ```csharp
    // Copy first
    string path = HttpContext.Request.Path;
    
    // Then parallelize
    await Task.WhenAll(
        DoWorkAsync(path),
        DoMoreWorkAsync(path)
    );
    ```

- [ ] **HttpContext not used after request completes**
  - [ ] No `async void` methods writing to response
  - [ ] All controller actions return `Task<ActionResult>`, not `ActionResult`
  - [ ] Background work uses `IServiceScopeFactory.CreateAsyncScope()`

- [ ] **Background services don't capture scoped dependencies**
  - [ ] No capturing `DbContext` from controller parameter in background tasks
  - [ ] Pattern:
    ```csharp
    [HttpPost("/work")]
    public IActionResult StartWork([FromServices] IServiceScopeFactory scopeFactory)
    {
        _ = Task.Run(async () =>
        {
            await using var scope = scopeFactory.CreateAsyncScope();
            var db = scope.ServiceProvider.GetRequiredService<DbContext>();
            // Use db in scope
        });
        return Accepted();
    }
    ```

---

## Phase 3: Request/Response Handling (CRITICAL)

> Synchronous reads block threads. Large buffering causes OutOfMemory issues.

### Checklist

- [ ] **Async APIs for request bodies**
  - [ ] No `new StreamReader(Request.Body).ReadToEnd()`
  - [ ] No `HttpContext.Request.Form` (sync access)
  - [ ] Use: `await Request.ReadFormAsync()`
  - [ ] Use: `await JsonSerializer.DeserializeAsync<T>(Request.Body)`

- [ ] **Async APIs for response bodies**
  - [ ] No `Response.WriteAsync(...)` awaited properly
  - [ ] Use: `await Response.WriteAsync(content)`
  - [ ] Use: `await JsonSerializer.SerializeAsync(Response.Body, obj)`

- [ ] **Large bodies not buffered entirely**
  - [ ] Warning if entire body read: `ReadAsStringAsync()`, `ReadToEndAsync()` on large files
  - [ ] Pattern: Stream or use chunked processing
  - [ ] JSON: Use `JsonSerializer.DeserializeAsync` (streaming)
  - [ ] Files: Use chunked reads with buffer

- [ ] **ContentLength properly handled**
  - [ ] Null checks for `Request.ContentLength`
  - [ ] Don't assume `ContentLength` is known
  - [ ] Example: `if (Request.ContentLength > 10_000_000) return BadRequest()`

---

## Phase 4: Performance & Caching (HIGH PRIORITY)

### Checklist

- [ ] **Large collections paginated**
  - [ ] No `ToListAsync()` without `Skip/Take`
  - [ ] Endpoints with `GetAll` return paginated results
  - [ ] Default page size <= 100
  - [ ] Pattern:
    ```csharp
    var users = await _context.Users
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    ```

- [ ] **IAsyncEnumerable for streaming (where appropriate)**
  - [ ] Large collections from APIs use `IAsyncEnumerable<T>`
  - [ ] Streaming endpoints return `async IAsyncEnumerable<T>`
  - [ ] Pagination preferred for APIs; streaming for exports

- [ ] **Caching strategy in place**
  - [ ] Frequently accessed data cached
  - [ ] Cache duration reasonable (15-60 minutes typically)
  - [ ] Cache invalidation on updates
  - [ ] Pattern:
    ```csharp
    if (_cache.TryGetValue(key, out var value))
        return Result.Success(value);
    // Fetch and cache...
    ```

- [ ] **No N+1 queries**
  - [ ] Related data uses `Include()`
  - [ ] No loops loading data: `foreach (var x in items) db.Related.Where(...).ToList()`
  - [ ] Single queries with JOINs
  - [ ] Command: Look for `.ToList()` or `.ToListAsync()` inside loops

- [ ] **AsNoTracking for read-only queries**
  - [ ] Read queries specify `.AsNoTracking()`
  - [ ] Write queries don't use AsNoTracking
  - [ ] Pattern:
    ```csharp
    var readOnlyUsers = await _context.Users
        .AsNoTracking()
        .Where(u => u.IsActive)
        .ToListAsync();
    ```

- [ ] **Select/project only needed columns**
  - [ ] `.Select(u => new { u.Id, u.Name })` for DTOs
  - [ ] Not `ToList()` then `.Select()`
  - [ ] Reduces network transfer and memory

- [ ] **DbContext pooling for high-scale apps**
  - [ ] Consider for apps with 1000+ req/sec
  - [ ] Configuration: `services.AddDbContextPool<Db>(..., poolSize: 128)`

- [ ] **IHttpClientFactory used for HTTP calls**
  - [ ] No `new HttpClient()` instances
  - [ ] Uses `_httpClientFactory.CreateClient()`
  - [ ] Prevents socket exhaustion

---

## Phase 5: Memory & Garbage Collection (MEDIUM)

### Checklist

- [ ] **Large allocations (>= 85KB) minimized in hot paths**
  - [ ] Warning: Large `byte[]` allocations in frequently-called code
  - [ ] Use `ArrayPool<byte>.Shared.Rent()` for temporary buffers
  - [ ] String concatenations in loops: Use `StringBuilder`

- [ ] **Proper disposal of resources**
  - [ ] `IDisposable` objects properly disposed
  - [ ] `using` or `await using` statements
  - [ ] No resource leaks from event handlers

- [ ] **GC pressure minimal**
  - [ ] Profile with PerfView if perf issues
  - [ ] Monitor: GC pause times, Gen2 collections
  - [ ] High LOH pressure indicates problem areas

---

## Phase 6: Error Handling (HIGH)

### Checklist

- [ ] **Result<T> pattern used for expected failures**
  - [ ] No throwing `ValidationException` in service layer
  - [ ] Returns `Result.Failure<T>("error message")`
  - [ ] Controllers handle `Result.IsFailure` → HTTP error codes

- [ ] **Exceptions minimized**
  - [ ] Exceptions for exceptional cases only
  - [ ] Not for control flow (if/else instead of try/catch)
  - [ ] Application Insights can flag exception rates

- [ ] **Meaningful error messages**
  - [ ] Errors include context
  - [ ] Example: `"User '123' not found"` not just `"Error"`
  - [ ] Client-facing errors don't expose internals

- [ ] **No sensitive data in responses**
  - [ ] Stack traces not returned to client
  - [ ] Internal exception messages not exposed
  - [ ] Generic error: "An error occurred" to external API consumers

---

## Phase 7: Security (CRITICAL)

### Checklist

- [ ] **No hardcoded secrets**
  - [ ] Connection strings from appsettings.json + env vars
  - [ ] API keys from KeyVault or secrets manager
  - [ ] Command: `grep -r "password\|secret\|key" src/ --include="*.cs"` (check results)

- [ ] **Input validation present**
  - [ ] All user inputs validated
  - [ ] Fluent Validation or similar
  - [ ] Database prevents SQL injection (EF Core parameterized)

- [ ] **Authentication/Authorization checked**
  - [ ] `[Authorize]` attributes on protected endpoints
  - [ ] Permission checks before operations
  - [ ] User context properly validated

- [ ] **No sensitive data in logs**
  - [ ] Passwords, tokens, PII not logged
  - [ ] Connection strings not logged
  - [ ] Command: `grep -r "LogInformation\|LogDebug" src/ | grep -i "password\|token\|secret"`

- [ ] **HTTPS in production**
  - [ ] Redirect HTTP to HTTPS
  - [ ] Secure cookies (`HttpOnly`, `Secure`, `SameSite`)

---

## Phase 8: Documentation & Code Quality (MEDIUM)

### Checklist

- [ ] **Public APIs documented**
  - [ ] XML doc comments on public types/methods
  - [ ] `<summary>`, `<param>`, `<returns>` present
  - [ ] Command: `dotnet build /p:GenerateDocumentationFile=true`

- [ ] **Complex logic commented**
  - [ ] Comments explain "why", not "what"
  - [ ] Code is self-documenting through clear naming
  - [ ] Non-obvious algorithms explained

- [ ] **Naming conventions followed**
  - [ ] Files: PascalCase matching class name
  - [ ] Classes: PascalCase
  - [ ] Methods: PascalCase with Async suffix
  - [ ] Private fields: `_camelCase`
  - [ ] Constants: `UPPER_SNAKE_CASE`

- [ ] **Methods under 30 lines**
  - [ ] Single Responsibility Principle
  - [ ] Cyclomatic complexity < 10
  - [ ] Consider refactoring if longer

- [ ] **DRY principle followed**
  - [ ] No code duplication
  - [ ] Shared logic extracted to methods
  - [ ] Patterns reused

---

## Phase 9: Testing (HIGH)

### Checklist

- [ ] **Unit tests pass**
  - [ ] `dotnet test` shows green
  - [ ] All tests execute successfully

- [ ] **Code coverage >= 80%**
  - [ ] `dotnet test /p:CollectCoverage=true`
  - [ ] Critical paths covered
  - [ ] Error paths tested

- [ ] **Happy path AND error paths tested**
  - [ ] Success scenarios covered
  - [ ] Failure scenarios covered
  - [ ] Result.Failure cases tested

- [ ] **Async methods properly tested**
  - [ ] Tests use `async Task`, not `void`
  - [ ] No `Task.Result` in tests
  - [ ] Mock async APIs return `Task` properly

- [ ] **Cancellation token support tested (if applicable)**
  - [ ] Methods accept `CancellationToken`
  - [ ] Cancellation tested

---

## Phase 10: Middleware & Startup (HIGH)

### Checklist

- [ ] **Middleware has no long-running tasks**
  - [ ] Middleware executes quickly
  - [ ] Early middleware is fastest
  - [ ] Long tasks → hosted services or message brokers

- [ ] **Dependency injection configured properly**
  - [ ] Services registered in DI container
  - [ ] Lifetimes correct (Singleton/Scoped/Transient)
  - [ ] No capturedHttpContext in background services

- [ ] **Error handling middleware in place**
  - [ ] Global exception handler
  - [ ] Errors logged properly
  - [ ] Responses formatted consistently

---

## Red Flags: Automatic Rejection

The following items require fixing before approval:

🔴 **BLOCKING ISSUES**:

- [ ] ❌ Any `.Result` or `.Wait()` in service/controller code
- [ ] ❌ Any `async void` methods (except event handlers)
- [ ] ❌ Build fails with warnings
- [ ] ❌ Tests don't pass
- [ ] ❌ Analyzers show CRITICAL issues
- [ ] ❌ HttpContext stored in field
- [ ] ❌ Synchronous `ReadToEnd()` or `Request.Form`
- [ ] ❌ Hardcoded secrets/credentials
- [ ] ❌ No CancellationToken on async methods
- [ ] ❌ `Task.WhenAll()` accessing HttpContext
- [ ] ❌ Entire collection loaded without pagination
- [ ] ❌ `async void` controller actions
- [ ] ❌ Entire request body buffered for large uploads

---

## Approval Checklist

✅ **Ready for Approval** when:

- [ ] All automated checks pass (build, lint, tests, coverage)
- [ ] No CRITICAL blocking issues found
- [ ] Async/await patterns correct (no blocking)
- [ ] HttpContext handled safely
- [ ] Request/response bodies handled properly
- [ ] Performance optimized (pagination, caching, no N+1)
- [ ] Error handling uses Result<T> pattern
- [ ] Security best practices followed
- [ ] Code quality and documentation acceptable
- [ ] Tests adequate

---

## Common Issues & How to Fix

### Issue: `.Result` or `.Wait()` found

```csharp
// ❌ WRONG
public Result<User> GetUser(string id)
{
    return _userService.GetUserAsync(id).Result;
}

// ✅ FIX
public async Task<Result<User>> GetUserAsync(string id)
{
    return await _userService.GetUserAsync(id);
}
```

### Issue: `async void` method

```csharp
// ❌ WRONG
public async void UpdateUser()
{
    await _service.UpdateAsync();
}

// ✅ FIX
public async Task UpdateUserAsync()
{
    await _service.UpdateAsync();
}
```

### Issue: All records returned without pagination

```csharp
// ❌ WRONG
public async Task<IEnumerable<User>> GetUsers()
{
    return await _context.Users.ToListAsync();
}

// ✅ FIX
public async Task<IActionResult> GetUsers([FromQuery] int page = 1, [FromQuery] int pageSize = 10)
{
    var total = await _context.Users.CountAsync();
    var users = await _context.Users
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return Ok(new { users, total, page, pageSize });
}
```

### Issue: HttpContext in background task

```csharp
// ❌ WRONG
_ = Task.Run(async () =>
{
    var path = HttpContext.Request.Path;
    await _service.DoWork(path);
});

// ✅ FIX
string path = HttpContext.Request.Path;
_ = Task.Run(async () =>
{
    await _service.DoWork(path);
});
```

### Issue: N+1 query problem

```csharp
// ❌ WRONG
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    user.Orders = await _context.Orders
        .Where(o => o.UserId == user.Id)
        .ToListAsync();
}

// ✅ FIX
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();
```

---

## For Architects Only: Design Review

When doing architecture review (not just code review), also check:

- [ ] System design follows layered architecture
- [ ] Domain/Application/Infrastructure separation
- [ ] Database schema normalized and indexed
- [ ] API contract versioning strategy
- [ ] Authentication/Authorization approach
- [ ] Caching layers at appropriate tiers
- [ ] Monitoring and observability in place
- [ ] Logging strategy (structure, levels, retention)
- [ ] Backup/Disaster recovery plan
- [ ] Scalability considered (horizontal scaling, bottlenecks)

---

## Quick Reference: Microsoft Best Practices Violations

| Violation | Severity | Fix | Reference |
|-----------|----------|-----|-----------|
| `.Result` / `.Wait()` | 🔴 CRITICAL | Use `await` | Thread pool starvation |
| `async void` | 🔴 CRITICAL | Use `async Task` | Unhandled exceptions crash app |
| HttpContext in field | 🔴 CRITICAL | Store `IHttpContextAccessor` | Wrong/null context |
| HttpContext from threads | 🔴 CRITICAL | Copy values first | Thread safety |
| Sync body read | 🔴 CRITICAL | Use `ReadFormAsync` | Thread pool starvation |
| No pagination | 🔴 CRITICAL | Limit results | OOM |
| N+1 queries | 🟠 HIGH | Use `Include()` | Performance |
| Large allocations in loop | 🟠 HIGH | Use `ArrayPool` | GC pressure |
| No caching | 🟠 HIGH | Add cache layer | Performance |
| `new HttpClient()` | 🟠 HIGH | Use `IHttpClientFactory` | Socket exhaustion |

