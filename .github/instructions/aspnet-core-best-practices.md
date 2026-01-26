---
applyTo: "**/*.cs"
description: "Microsoft ASP.NET Core 10 Best Practices - Architectural Review"
---

# Microsoft ASP.NET Core 10 Best Practices

> This guide incorporates official Microsoft ASP.NET Core best practices for performance, security, and reliability.
> Reference: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices

## 🔴 CRITICAL: Avoid Blocking Calls

**Blocking calls are the #1 performance issue in ASP.NET Core apps.** They cause thread pool starvation and degraded response times.

### Thread Pool Starvation

When you block a thread, you prevent it from handling other requests:

```csharp
// ❌ CRITICAL ISSUE: Thread pool starvation
public Result<User> GetUser(string userId)
{
    return _repository.GetByIdAsync(userId).Result; // Blocks thread pool thread!
}

// If many requests do this:
// 1. Thread pool threads get blocked
// 2. New threads can't be created fast enough
// 3. Request queue grows
// 4. App becomes unresponsive
// 5. Eventually OutOfMemoryException
```

### Do Not Block Asynchronous Execution

```csharp
// ❌ DON'T: Any of these
public async Task<Result<User>> GetUserAsync(string userId)
{
    return _repository.GetByIdAsync(userId).Result;        // BLOCK
    return _repository.GetByIdAsync(userId).Wait() as Result; // BLOCK
    Task.Run(() => GetUser(userId)).Result;               // WASTE + BLOCK
}

// ✅ DO: Always await
public async Task<Result<User>> GetUserAsync(string userId)
{
    return await _repository.GetByIdAsync(userId);
}

// ✅ DO: Make controller actions async
[HttpGet("{id}")]
public async Task<ActionResult<UserDto>> GetUser(string id) // async + Task<T>
{
    var result = await _userService.GetUserAsync(id);
    return result.IsSuccess ? Ok(result.Value) : NotFound();
}

// ❌ DON'T: Synchronous controller actions
[HttpGet("{id}")]
public ActionResult<UserDto> GetUser(string id) // Entire call stack not async!
{
    var result = _userService.GetUser(id);
    return Ok(result);
}
```

### Never Use Task.Run in Hot Code Paths

```csharp
// ❌ DON'T: Task.Run is wasteful
public async Task<Result<User>> GetUserAsync(string userId)
{
    // ASP.NET Core already runs on thread pool
    // Task.Run schedules ANOTHER thread pool work item
    // This wastes thread pool capacity!
    return await Task.Run(() => _repository.GetUser(userId));
}

// ✅ DO: Call async API directly
public async Task<Result<User>> GetUserAsync(string userId)
{
    return await _repository.GetByIdAsync(userId);
}
```

## HttpContext Lifetime & Thread Safety

**HttpContext is request-scoped and NOT thread-safe.** Incorrect usage causes crashes or data corruption.

### Never Store HttpContext

```csharp
// ❌ DANGEROUS: Stores wrong or null context
public class BadUserService
{
    private readonly HttpContext _context; // WRONG!
    
    public BadUserService(IHttpContextAccessor accessor)
    {
        _context = accessor.HttpContext; // Null or from wrong request
    }
    
    public void CheckAdmin()
    {
        if (!_context.User.IsInRole("admin")) // Wrong user!
            throw new Exception();
    }
}

// ✅ SAFE: Store IHttpContextAccessor
public class GoodUserService
{
    private readonly IHttpContextAccessor _accessor;
    
    public GoodUserService(IHttpContextAccessor accessor)
    {
        _accessor = accessor;
    }
    
    public void CheckAdmin()
    {
        var context = _accessor.HttpContext; // Get current context
        if (context != null && !context.User.IsInRole("admin"))
            throw new Exception();
    }
}
```

### Never Access HttpContext from Multiple Threads

```csharp
// ❌ DANGEROUS: HttpContext accessed from parallel tasks
public async Task<SearchResults> BadParallelSearch(string query)
{
    var task1 = SearchAsync(SearchEngine.Google, query); // Logs using HttpContext.Request.Path
    var task2 = SearchAsync(SearchEngine.Bing, query);   // From different thread!
    var task3 = SearchAsync(SearchEngine.DuckDuckGo, query);
    
    await Task.WhenAll(task1, task2, task3);
    
    private async Task<SearchResults> SearchAsync(SearchEngine engine, string query)
    {
        try
        {
            _logger.LogInformation("Path: {path}", HttpContext.Request.Path); // THREAD SAFETY ISSUE!
            var results = await _searchService.SearchAsync(engine, query);
            _logger.LogInformation("Path: {path}", HttpContext.Request.Path); // WRONG PATH!
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Path: {path}", HttpContext.Request.Path); // CRASH!
        }
        return results;
    }
}

// ✅ SAFE: Copy values before parallelization
public async Task<SearchResults> GoodParallelSearch(string query)
{
    string path = HttpContext.Request.Path; // Copy BEFORE Task.WhenAll
    
    var task1 = SearchAsync(SearchEngine.Google, query, path);
    var task2 = SearchAsync(SearchEngine.Bing, query, path);
    var task3 = SearchAsync(SearchEngine.DuckDuckGo, query, path);
    
    await Task.WhenAll(task1, task2, task3);
    
    private async Task<SearchResults> SearchAsync(SearchEngine engine, string query, string path)
    {
        try
        {
            _logger.LogInformation("Path: {path}", path); // Safe - copied value
            var results = await _searchService.SearchAsync(engine, query);
            _logger.LogInformation("Path: {path}", path); // Safe - copied value
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Path: {path}", path); // Safe - copied value
        }
        return results;
    }
}
```

### Never Use HttpContext After Request Completes

```csharp
// ❌ CRITICAL: async void completes after first await
public async void BadFireAndForget() // ← async void is ALWAYS BAD!
{
    await Task.Delay(1000);
    
    // HttpContext is recycled here - process will crash!
    await Response.WriteAsync("Hello World");
}

// ✅ SAFE: Always return Task
public async Task GoodFireAndForget()
{
    await Task.Delay(1000);
    
    // HttpContext still valid until Task completes
    await Response.WriteAsync("Hello World");
}

// If you need background work, use hosted services or IServiceScopeFactory
[HttpPost("/async-work")]
public IActionResult StartBackgroundWork([FromServices] IServiceScopeFactory scopeFactory)
{
    _ = Task.Run(async () =>
    {
        await Task.Delay(1000);
        
        // Create a new scope - don't capture request context
        await using var scope = scopeFactory.CreateAsyncScope();
        var service = scope.ServiceProvider.GetRequiredService<IMyService>();
        await service.DoWorkAsync();
    });
    
    return Accepted();
}
```

## Request/Response Body Handling

**Always use async APIs for reading/writing request and response bodies.**

### Synchronous Reads Cause Thread Pool Starvation

```csharp
// ❌ DON'T: Synchronous read (sync-over-async)
public IActionResult BadRead()
{
    var json = new StreamReader(Request.Body).ReadToEnd(); // BLOCKS THREAD!
    return Ok(JsonSerializer.Deserialize<ContosoData>(json));
}

// ✅ DO: Async read
public async Task<IActionResult> GoodRead()
{
    var data = await JsonSerializer.DeserializeAsync<ContosoData>(Request.Body);
    return Ok(data);
}

// ❌ DON'T: Sync form read (sync-over-async)
public IActionResult BadFormRead()
{
    var form = HttpContext.Request.Form; // Blocks!
    Process(form["id"], form["name"]);
    return Accepted();
}

// ✅ DO: Async form read
public async Task<IActionResult> GoodFormRead()
{
    var form = await HttpContext.Request.ReadFormAsync();
    Process(form["id"], form["name"]);
    return Accepted();
}
```

### Avoid Reading Large Bodies Into Memory

**Large allocations (>= 85KB) go to Large Object Heap (LOH) and trigger expensive Gen2 garbage collection.**

```csharp
// ❌ DANGEROUS: Reads entire body into memory
public async Task<IActionResult> UnsafeUpload()
{
    var body = await Request.Body.ReadAsStringAsync(); // 100MB file = OOM!
    return Ok();
}

// ✅ SAFE: Stream the body
public async Task<IActionResult> SafeUpload()
{
    // Let JsonSerializer handle streaming
    var data = await JsonSerializer.DeserializeAsync<UploadData>(Request.Body);
    return Ok();
}

// ✅ SAFE: Process in chunks
public async Task<IActionResult> ChunkedUpload()
{
    const int chunkSize = 1024 * 64; // 64KB chunks
    byte[] buffer = new byte[chunkSize];
    int bytesRead;
    
    while ((bytesRead = await Request.Body.ReadAsync(buffer, 0, chunkSize)) > 0)
    {
        await ProcessChunkAsync(buffer, bytesRead);
    }
    
    return Ok();
}
```

## Caching Strategy

**Cache aggressively to reduce database/external service load.**

### Memory Caching

```csharp
public class UserService
{
    private readonly IMemoryCache _cache;
    private readonly IUserRepository _repository;
    private const string UserCacheKey = "user_{0}";
    private const int CacheDurationMinutes = 15;
    
    public async Task<Result<User>> GetUserAsync(string userId)
    {
        var cacheKey = string.Format(UserCacheKey, userId);
        
        // Try cache first
        if (_cache.TryGetValue(cacheKey, out User cachedUser))
            return Result.Success(cachedUser);
        
        // Cache miss - fetch from DB
        var result = await _repository.GetByIdAsync(userId);
        
        // Cache success
        if (result.IsSuccess)
        {
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetAbsoluteExpiration(TimeSpan.FromMinutes(CacheDurationMinutes));
            _cache.Set(cacheKey, result.Value, cacheOptions);
        }
        
        return result;
    }
}
```

### Distributed Caching for Scale

```csharp
public class ScalableUserService
{
    private readonly IDistributedCache _cache;
    private readonly IUserRepository _repository;
    
    public async Task<Result<User>> GetUserAsync(string userId, CancellationToken cancellationToken = default)
    {
        var cacheKey = $"user_{userId}";
        var cachedJson = await _cache.GetStringAsync(cacheKey, token: cancellationToken);
        
        if (!string.IsNullOrEmpty(cachedJson))
        {
            var user = JsonSerializer.Deserialize<User>(cachedJson);
            return Result.Success(user!);
        }
        
        // Cache miss
        var result = await _repository.GetByIdAsync(userId, cancellationToken);
        if (result.IsSuccess)
        {
            var json = JsonSerializer.Serialize(result.Value);
            await _cache.SetStringAsync(cacheKey, json,
                new DistributedCacheEntryOptions()
                    .SetAbsoluteExpiration(TimeSpan.FromMinutes(15)),
                token: cancellationToken);
        }
        
        return result;
    }
}
```

## Database Optimization

### Return Large Collections as Paginated Results

**Never return all records at once. This causes memory issues and slow responses.**

```csharp
// ❌ DON'T: Return everything
[HttpGet]
public async Task<ActionResult<IEnumerable<User>>> GetAllUsers()
{
    var users = await _context.Users.ToListAsync(); // 1M records = OOM!
    return Ok(users);
}

// ✅ DO: Return paginated results
[HttpGet]
public async Task<ActionResult<ApiResponse<IEnumerable<UserDto>>>> GetUsers(
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 10,
    CancellationToken cancellationToken = default)
{
    if (pageSize > 100) pageSize = 100; // Max 100 per page
    
    var total = await _context.Users.CountAsync(cancellationToken);
    var users = await _context.Users
        .AsNoTracking()
        .OrderBy(u => u.Id)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(u => new UserDto(u.Id, u.Name))
        .ToListAsync(cancellationToken);
    
    return Ok(ApiResponse.Success(users, new { total, page, pageSize }));
}

// ✅ DO: Use IAsyncEnumerable for streaming (when appropriate)
[HttpGet("stream")]
public async IAsyncEnumerable<UserDto> GetUsersStreaming(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    var pageSize = 100;
    var page = 0;
    
    while (true)
    {
        var users = await _context.Users
            .AsNoTracking()
            .OrderBy(u => u.Id)
            .Skip(page * pageSize)
            .Take(pageSize)
            .Select(u => new UserDto(u.Id, u.Name))
            .ToListAsync(cancellationToken);
        
        if (users.Count == 0) yield break;
        
        foreach (var user in users)
            yield return user;
        
        page++;
    }
}
```

### Avoid N+1 Queries

```csharp
// ❌ DON'T: N+1 queries
public async Task<List<UserWithOrders>> GetUsersWithOrdersAsync()
{
    var users = await _context.Users.ToListAsync();
    var result = new List<UserWithOrders>();
    
    foreach (var user in users)
    {
        var orders = await _context.Orders.Where(o => o.UserId == user.Id).ToListAsync();
        // 1 + N queries!
        result.Add(new UserWithOrders(user, orders));
    }
    
    return result;
}

// ✅ DO: Use Include
public async Task<List<UserWithOrders>> GetUsersWithOrdersAsync()
{
    return await _context.Users
        .AsNoTracking()
        .Include(u => u.Orders)
        .Select(u => new UserWithOrders(u, u.Orders))
        .ToListAsync();
    // Single query with JOIN
}
```

### Use AsNoTracking for Read-Only Queries

```csharp
// ✅ DO: AsNoTracking for reads
var users = await _context.Users
    .AsNoTracking() // Don't track changes - faster
    .Where(u => u.IsActive)
    .ToListAsync();

// ✅ DO: Project only needed fields
var userEmails = await _context.Users
    .AsNoTracking()
    .Select(u => new { u.Id, u.Email }) // Only 2 columns
    .ToListAsync();
```

## HttpClientFactory Connection Pooling

**Never create new HttpClient instances - they exhaust socket pool.**

```csharp
// ❌ DON'T: Create new clients
public class BadHttpService
{
    public async Task<T> GetAsync<T>(string url)
    {
        using var client = new HttpClient(); // New instance each call!
        // Leaves socket in TIME_WAIT state
        // Eventually exhausts socket pool
        var response = await client.GetAsync(url);
        return await response.Content.ReadAsAsync<T>();
    }
}

// ✅ DO: Use IHttpClientFactory
public class GoodHttpService
{
    private readonly IHttpClientFactory _httpClientFactory;
    
    public GoodHttpService(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }
    
    public async Task<T> GetAsync<T>(string url)
    {
        var client = _httpClientFactory.CreateClient();
        var response = await client.GetAsync(url);
        return await response.Content.ReadAsAsync<T>();
    }
}

// In Startup/Program.cs:
// services.AddHttpClient();
// services.AddHttpClient<IMyApiClient, MyApiClient>();
```

## Minimize Exceptions

**Exceptions are slow. Use Result<T> pattern for expected failures.**

```csharp
// ❌ DON'T: Exceptions for control flow
public Result<User> GetUser(string userId)
{
    try
    {
        if (string.IsNullOrEmpty(userId))
            throw new ValidationException("User ID required"); // Slow!
        
        var user = _repository.Find(userId);
        if (user is null)
            throw new NotFoundException("User not found"); // Slow!
        
        return Result.Success(user);
    }
    catch (ValidationException ex)
    {
        return Result.Failure<User>(ex.Message);
    }
    catch (NotFoundException ex)
    {
        return Result.Failure<User>(ex.Message);
    }
}

// ✅ DO: Use Result<T> pattern
public Result<User> GetUser(string userId)
{
    if (string.IsNullOrEmpty(userId))
        return Result.Failure<User>("User ID required");
    
    var user = _repository.Find(userId);
    return user is not null
        ? Result.Success(user)
        : Result.Failure<User>("User not found");
}
```

## Response Compression

**Enable response compression to reduce bandwidth.**

```csharp
// In Program.cs / Startup:
builder.Services.AddResponseCompression(options =>
{
    options.Providers.Add<GzipCompressionProvider>();
    options.Providers.Add<BrotliCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.Fastest;
});

var app = builder.Build();
app.UseResponseCompression();
```

## Key Performance Metrics

Monitor these metrics to identify issues:

| Metric | Target | Tool |
|--------|--------|------|
| Request Latency | < 200ms | Application Insights, PerfView |
| Thread Pool Threads | Stable | PerfView, Diagnostics |
| GC Pause Time | < 50ms | PerfView, dotTrace |
| Memory Usage | Stable | Application Insights, dotMemory |
| DB Query Time | < 100ms | Database profiler, Application Insights |
| Cache Hit Rate | > 80% | Application Insights |

## Checklist for Architects/Reviewers

When reviewing ASP.NET Core code, ensure:

```
Async/Await:
☐ All I/O operations are async (database, HTTP, streams, forms)
☐ No Task.Result, Task.Wait(), or sync-over-async patterns
☐ No async void methods (except event handlers)
☐ Controller actions return Task<ActionResult>
☐ CancellationToken support throughout

HttpContext:
☐ No HttpContext stored in fields
☐ Values copied before parallel operations (Task.WhenAll, etc.)
☐ No access from multiple threads
☐ No use after request completes
☐ Background work uses IServiceScopeFactory or hosted services

Request/Response Handling:
☐ Async APIs used for body reads (ReadFormAsync, DeserializeAsync, etc.)
☐ Large bodies streamed, not buffered
☐ ContentLength null-checked properly
☐ Form data read with ReadFormAsync, not Request.Form

Performance:
☐ Large collections paginated
☐ Caching strategy in place
☐ No N+1 queries
☐ AsNoTracking used for read-only queries
☐ IHttpClientFactory used for HTTP calls
☐ Response compression enabled
☐ Profiling done on hot code paths

Memory:
☐ Large allocations (>= 85KB) minimized in hot paths
☐ ArrayPool used for temporary buffers
☐ Objects disposed properly
☐ No memory leaks

Error Handling:
☐ Result<T> pattern used for expected failures
☐ Exceptions minimized
☐ Meaningful error messages
☐ No sensitive data in responses or logs
```

## References

- [Microsoft ASP.NET Core Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)
- [async/await Best Practices in C#](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [Garbage Collection Performance](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/performance)
- [Entity Framework Core Performance](https://learn.microsoft.com/en-us/ef/core/performance/)
- [Caching Overview](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/overview)
