# Quality Standards & Automated Checks

> This document defines the quality gates and automated checks that agents must validate during code review and development.
> This template implements [Microsoft ASP.NET Core Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)
> For architectural reviews, see [ASP.NET Core Best Practices Guide](.github/instructions/aspnet-core-best-practices.md)

## Overview

Quality standards are organized by category and enforcement level. Agents should verify each standard before passing code forward.

## Enforcement Levels

- **CRITICAL** 🔴: Must pass - code cannot proceed without fixing
- **HIGH** 🟠: Should pass - must have justification to skip
- **MEDIUM** 🟡: Recommended - should fix unless specific reason not to
- **LOW** 🔵: Nice to have - improve if time permits

## Code Quality

### Compilation & Build

| Standard | Enforcement | Check | Command |
|----------|-------------|-------|---------|
| Code compiles without errors | CRITICAL 🔴 | Zero build errors | `dotnet build` |
| No compiler warnings | CRITICAL 🔴 | TreatWarningsAsErrors = true | `dotnet build` |
| Static analysis passes | CRITICAL 🔴 | All analyzers green | `dotnet build` |

### Static Analysis Checklist

**SonarAnalyzer.CSharp** 🔴 CRITICAL
- [ ] No code smells (complexity, duplication, maintainability)
- [ ] No security vulnerabilities identified
- [ ] No potential bugs detected
- Run: `dotnet build /p:TreatWarningsAsErrors=true`

**Microsoft.CodeAnalysis.NetAnalyzers** 🔴 CRITICAL
- [ ] API usage follows best practices
- [ ] No deprecated API usage
- [ ] Proper exception handling
- [ ] Async/await usage correct
- Violations: Build fails

**Meziantou.Analyzer** 🔴 CRITICAL
- [ ] Using modern C# features appropriately
- [ ] String formatting optimized
- [ ] LINQ usage efficient
- [ ] Regular expressions validated
- Violations: Build fails

### Code Structure

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Methods under 30 lines | HIGH 🟠 | Count lines, consider refactoring |
| Single Responsibility Principle | HIGH 🟠 | Each method has one clear purpose |
| Max 3-4 method parameters | MEDIUM 🟡 | Use options object for complex signatures |
| DRY principle followed | HIGH 🟠 | No code duplication - refactor |
| Cyclomatic complexity < 10 | MEDIUM 🟡 | Consider breaking down complex methods |
| Proper error handling | HIGH 🟠 | Use Result<T> pattern, no silent failures |

### Naming Conventions

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Files PascalCase | CRITICAL 🔴 | Match class name: `UserService.cs` |
| Classes PascalCase | CRITICAL 🔴 | `class UserService` |
| Methods PascalCase | CRITICAL 🔴 | `public async Task GetUserAsync()` |
| Properties PascalCase | CRITICAL 🔴 | `public string FirstName { get; set; }` |
| Private fields _ prefix | CRITICAL 🔴 | `private IRepository _repository` |
| Constants UPPER_SNAKE_CASE | CRITICAL 🔴 | `const int MAX_RETRY_COUNT = 5` |
| Async methods Async suffix | HIGH 🟠 | `GetUserAsync`, `CreateUserAsync` |
| Boolean methods Is/Has/Can | MEDIUM 🟡 | `IsActive`, `HasPermission`, `CanDelete` |

## Nullable Reference Types & Type Safety

| Standard | Enforcement | Check |
|----------|-------------|-------|
| #nullable enable in all files | CRITICAL 🔴 | Enabled at file or project level |
| No `#pragma` null suppress | HIGH 🟠 | Address null warnings, don't suppress |
| No `nullable: disable` | CRITICAL 🔴 | All code must use strict null checks |
| Proper null annotations | HIGH 🟠 | String vs string? used correctly |
| No `[AllowNull]` misuse | MEDIUM 🟡 | Document when properties can be null |

## Railway-Oriented Programming Pattern

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Operations return Result<T> | HIGH 🟠 | Methods that can fail use Result pattern |
| No exceptions for expected failures | HIGH 🟠 | Use Result.Failure(), not throw |
| Proper Result chaining | HIGH 🟠 | Use Map, Bind, Tap correctly |
| Async operations use MapAsync/BindAsync | HIGH 🟠 | Never block with .Result or .Wait() |

## Documentation

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Public types have XML docs | HIGH 🟠 | `///` comments on all public classes/interfaces |
| Public methods have XML docs | HIGH 🟠 | `///` with `<summary>`, `<param>`, `<returns>` |
| Parameters documented | HIGH 🟠 | `<param name="x">Description</param>` |
| Return values documented | HIGH 🟠 | `<returns>Description</returns>` |
| Exceptions documented | MEDIUM 🟡 | `<exception cref="Ex">When thrown</exception>` |
| Complex logic has comments | MEDIUM 🟡 | Explain "why", not "what" |
| XML docs generate without warnings | HIGH 🟠 | Run with `/p:GenerateDocumentationFile=true` |

### XML Documentation Template

```csharp
/// <summary>
/// Brief description of what this method does.
/// </summary>
/// <param name="userId">The unique identifier of the user.</param>
/// <param name="cancellationToken">Token to cancel the operation.</param>
/// <returns>
/// A <see cref="Result{T}"/> containing the user if found,
/// or a failure result if not found or if input is invalid.
/// </returns>
/// <exception cref="ArgumentNullException">Thrown when userId is null.</exception>
/// <example>
/// <code>
/// var result = await service.GetUserAsync("123");
/// if (result.IsSuccess)
///     Console.WriteLine(result.Value.Name);
/// </code>
/// </example>
public async Task<Result<User>> GetUserAsync(string userId, CancellationToken cancellationToken = default)
{
    // implementation
}
```

## Async/Await Compliance

| Standard | Enforcement | Check |
|----------|-------------|-------|
| All I/O operations async | CRITICAL 🔴 | Database, HTTP, file system all async |
| Methods return Task/Task<T> | CRITICAL 🔴 | Not void (except event handlers) |
| No .Result or .Wait() usage | CRITICAL 🔴 | Never block - causes deadlocks |
| ConfigureAwait(false) in libraries | HIGH 🟠 | Library code uses ConfigureAwait(false) |
| Proper CancellationToken support | HIGH 🟠 | I/O operations accept CancellationToken |
| Async IEnumerable for streaming | HIGH 🟠 | Use `async IAsyncEnumerable` for streams |

## Testing

| Standard | Enforcement | Check | Coverage |
|----------|-------------|-------|----------|
| Minimum code coverage | HIGH 🟠 | >= 80% overall | `dotnet test /p:CollectCoverage=true` |
| All critical paths tested | HIGH 🟠 | Success and failure paths covered | 95%+ for critical services |
| Edge cases tested | HIGH 🟠 | null, empty, boundary values | 100% for value objects |
| Error scenarios tested | HIGH 🟠 | All Result.Failure cases tested | 100% for error handling |
| Test names descriptive | MEDIUM 🟡 | `MethodName_Scenario_ExpectedResult` | All tests |
| No skipped tests | MEDIUM 🟡 | All `.Skip` tests have issue reference | 0 skipped |
| Integration tests for APIs | MEDIUM 🟡 | E2E tests for critical flows | At least 50% |
| Async tests properly await | HIGH 🟠 | No race conditions or incomplete assertions | All async tests |

### Test Quality Checklist

```
Unit Tests:
☐ Follows AAA pattern (Arrange, Act, Assert)
☐ Single logical assertion per test
☐ Test name clearly describes scenario
☐ Uses appropriate mock/stub strategy
☐ Cleans up resources after test

Integration Tests:
☐ Uses test database or in-memory database
☐ Transaction isolation for each test
☐ Realistic test data setup
☐ Validates end-to-end flow
☐ Cleans up test data after run
```

## Performance

| Standard | Enforcement | Check |
|----------|-------------|-------|
| No N+1 queries in loops | HIGH 🟠 | Use Include/Join for related data |
| Efficient pagination | MEDIUM 🟡 | Skip/Take with indexes |
| String operations optimized | MEDIUM 🟡 | StringBuilder for loops, string interpolation |
| LINQ queries efficient | MEDIUM 🟡 | Execute server-side when possible |
| Appropriate async operations | HIGH 🟠 | Use async for I/O, not CPU-bound work |
| Memory leaks prevented | HIGH 🟠 | Proper disposal of resources |
| API response time acceptable | MEDIUM 🟡 | < 200ms for typical endpoints |

## Security

| Standard | Enforcement | Check |
|----------|-------------|-------|
| No hardcoded secrets | CRITICAL 🔴 | Use appsettings.json + environment variables |
| Input validation present | HIGH 🟠 | All user inputs validated and sanitized |
| SQL injection prevention | CRITICAL 🔴 | Use EF Core (parameterized) |
| Authentication required | HIGH 🟠 | Proper [Authorize] attributes |
| Authorization checks | HIGH 🟠 | Verify permissions before operations |
| No sensitive data in logs | HIGH 🟠 | Never log passwords, tokens, PII |
| HTTPS in production | CRITICAL 🔴 | All endpoints HTTPS-only |
| Dependencies updated | HIGH 🟠 | No known CVEs: `dotnet list package --vulnerable` |

## Error Handling

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Result<T> pattern used | HIGH 🟠 | Expected failures return Result.Failure |
| Specific error types | HIGH 🟠 | Custom error types, not generic Exception |
| Meaningful error messages | MEDIUM 🟡 | Include context about what failed |
| Errors don't leak details | HIGH 🟠 | Don't expose internal errors to client |
| Proper exception catching | HIGH 🟠 | Catch specific exceptions, not base Exception |
| Logging on errors | HIGH 🟠 | Errors logged with sufficient context |

## Code Review Checklist

### Before Passing to Next Phase

```
Syntax & Compilation:
☐ Code compiles without errors
☐ No compiler warnings (TreatWarningsAsErrors passes)
☐ SonarAnalyzer.CSharp: PASS
☐ Microsoft.CodeAnalysis.NetAnalyzers: PASS
☐ Meziantou.Analyzer: PASS

Code Quality:
☐ Naming conventions followed
☐ Methods under 30 lines
☐ Single Responsibility Principle
☐ DRY principle followed
☐ Proper error handling (Result<T> pattern)

Documentation:
☐ Public types have XML docs
☐ Methods have [summary], [param], [returns]
☐ Complex logic has explanatory comments
☐ No TODOs without issue reference

Testing:
☐ Unit tests pass
☐ Code coverage >= 80%
☐ Edge cases covered
☐ Error paths tested

Performance:
☐ No N+1 queries
☐ Appropriate async/await
☐ Efficient data structures

Security:
☐ No hardcoded secrets
☐ Input validation present
☐ No sensitive data in logs
☐ Authorization checks

Git:
☐ Commits follow conventional format
☐ Commit messages descriptive
☐ Changes are cohesive
```

## Automated Quality Gates

### Local Development

```bash
# Format and check
dotnet format

# Build with strict quality
dotnet build /p:TreatWarningsAsErrors=true

# Run all tests with coverage
dotnet test /p:CollectCoverage=true /p:CoverageFormat=opencover

# Check for vulnerable dependencies
dotnet list package --vulnerable
```

### CI/CD Pipeline

All of the above must pass before code can be merged.


☐ Functions under 30 lines
☐ No code duplication
☐ Naming conventions followed
☐ Error handling in place

Documentation:
☐ Public functions have JSDoc
☐ Parameters documented
☐ Return values documented
☐ Complex logic has comments

Performance:
☐ No N+1 queries
☐ No blocking operations
☐ Reasonable memory usage

Security:
☐ No hardcoded secrets
☐ Input validation present
☐ No SQL injection risk
☐ Permissions checked

Testing:
☐ 80%+ code coverage
☐ Critical paths tested
☐ Edge cases tested
☐ Error cases tested

Other:
☐ Type definitions complete
☐ No dead code
☐ README updated if needed
```

## Quality Tools

| Tool | Purpose | Command |
|------|---------|---------|
| ESLint | Code linting | `npm run lint` |
| Prettier | Code formatting | `npm run format` |
| TypeScript | Type checking | `npm run check-types` |
| Jest/Vitest | Unit testing | `npm run test` |
| Coverage | Code coverage | `npm run coverage` |
| npm audit | Dependency security | `npm audit` |
| SonarQube | Code analysis | CI/CD integration |

## Continuous Improvement

When agents identify new patterns or standards:

1. Document in this file
2. Add to improvement-patterns.md if it's a common issue
3. Reference in agent-review-checklist.md
4. Update team on changes
