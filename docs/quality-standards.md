# Quality Standards & Automated Checks

> This document defines the quality gates and automated checks that agents must validate during code review and development.

## Overview

Quality standards are organized by category and enforcement level. Agents should verify each standard before passing code forward.

## Enforcement Levels

- **CRITICAL** 🔴: Must pass - code cannot proceed without fixing
- **HIGH** 🟠: Should pass - must have justification to skip
- **MEDIUM** 🟡: Recommended - should fix unless specific reason not to
- **LOW** 🔵: Nice to have - improve if time permits

## Code Quality

### Syntax & Compilation

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Code compiles/runs without errors | CRITICAL 🔴 | `npm run build` or language-specific build |
| No TypeScript/type errors | CRITICAL 🔴 | Type checking enabled, no `any` without justification |
| ESLint passes with no errors | CRITICAL 🔴 | `npm run lint` - all errors fixed |
| Code formatting consistent | CRITICAL 🔴 | `npm run format` - uses project formatter |

### Code Structure

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Functions under 30 lines | HIGH 🟠 | Count lines, consider refactoring |
| Single Responsibility Principle | HIGH 🟠 | Each function has one clear purpose |
| Max 3-4 function parameters | MEDIUM 🟡 | Use options object for complex signatures |
| DRY principle followed | HIGH 🟠 | No code duplication - refactor repeated logic |
| Cyclomatic complexity < 10 | MEDIUM 🟡 | Consider breaking down complex functions |
| Proper error handling | HIGH 🟠 | No silent failures, explicit error types |

### Naming Conventions

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Files in kebab-case | CRITICAL 🔴 | File names follow `my-file.ts` pattern |
| Classes in PascalCase | CRITICAL 🔴 | `class UserService` |
| Functions in camelCase | CRITICAL 🔴 | `function getUserById()` |
| Constants in UPPER_SNAKE_CASE | CRITICAL 🔴 | `const MAX_RETRIES = 5` |
| Boolean functions start with is/has | MEDIUM 🟡 | `isActive()`, `hasPermission()` |
| Semantic naming | HIGH 🟠 | Names describe purpose, avoid ambiguous names |

## Documentation

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Public functions have JSDoc/docstring | HIGH 🟠 | All exported functions documented |
| Parameters documented | HIGH 🟠 | `@param` tags with type and description |
| Return values documented | HIGH 🟠 | `@returns` tag with type and description |
| Exceptions documented | MEDIUM 🟡 | `@throws` tags for thrown errors |
| Complex logic has comments | MEDIUM 🟡 | "Why" comments on non-obvious code |
| README updated for new features | MEDIUM 🟡 | User-facing changes in docs |
| Type definitions documented | MEDIUM 🟡 | Complex types have comments |

**JSDoc Template**:
```typescript
/**
 * Brief description of what this does.
 * 
 * Longer description if needed, explaining context and behavior.
 * 
 * @param name - Description of parameter
 * @param options - Configuration options
 * @returns Description of return value
 * @throws {ErrorType} When this error occurs
 * 
 * @example
 * const result = await myFunction('value', { flag: true });
 */
```

## Performance

| Standard | Enforcement | Check |
|----------|-------------|-------|
| No N+1 queries in loops | HIGH 🟠 | Batch database queries |
| Avoid unnecessary re-renders (React) | HIGH 🟠 | Proper memoization and deps |
| Appropriate data structure usage | MEDIUM 🟡 | Use Set/Map for lookups, Array for iteration |
| No blocking operations in main thread | HIGH 🟠 | Async/await for I/O operations |
| Reasonable memory usage | MEDIUM 🟡 | No memory leaks from closures/timers |
| API response time < 200ms | MEDIUM 🟡 | Acceptable latency for user-facing endpoints |

## Security

| Standard | Enforcement | Check |
|----------|-------------|-------|
| No hardcoded secrets | CRITICAL 🔴 | Use environment variables |
| Input validation on all user input | HIGH 🟠 | Sanitize and validate all inputs |
| No SQL injection vulnerabilities | CRITICAL 🔴 | Use parameterized queries |
| Proper authentication checks | HIGH 🟠 | Verify user permissions before operations |
| No XSS vulnerabilities (web) | HIGH 🟠 | Proper output encoding |
| Dependencies without known CVEs | HIGH 🟠 | Run `npm audit` |
| No sensitive data in logs | HIGH 🟠 | Don't log passwords, tokens, PII |

## Testing

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Minimum 80% code coverage | HIGH 🟠 | Coverage report shows >= 80% |
| All critical paths tested | HIGH 🟠 | Happy path and error paths |
| Edge cases tested | HIGH 🟠 | null/undefined, empty arrays, boundary values |
| Error scenarios tested | HIGH 🟠 | All throw statements tested |
| Test names describe behavior | MEDIUM 🟡 | `test_getUserById_returnsUserWhenFound` |
| No skipped tests | MEDIUM 🟡 | All `.skip` tests have issue reference |
| Integration tests for APIs | MEDIUM 🟡 | E2E tests for critical flows |
| Async tests properly await | HIGH 🟠 | No race conditions in tests |

## Error Handling

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Specific error types thrown | HIGH 🟠 | Not generic Error, use ValidationError, NotFoundError, etc. |
| Error messages are descriptive | MEDIUM 🟡 | Include context about what failed |
| Errors propagate appropriately | HIGH 🟠 | Don't swallow errors silently |
| Try-catch at right level | HIGH 🟠 | Catch at service layer, let caller handle |
| Graceful degradation | MEDIUM 🟡 | System doesn't crash on errors |

## Accessibility (if web application)

| Standard | Enforcement | Check |
|----------|-------------|-------|
| ARIA labels on interactive elements | MEDIUM 🟡 | Buttons, inputs have proper labels |
| Keyboard navigation support | MEDIUM 🟡 | All features usable with keyboard |
| Color contrast sufficient | MEDIUM 🟡 | WCAG AA compliance for color contrast |
| Images have alt text | MEDIUM 🟡 | Descriptive alt text for all images |

## Git & Version Control

| Standard | Enforcement | Check |
|----------|-------------|-------|
| Commits follow conventional format | MEDIUM 🟡 | `feat:`, `fix:`, `docs:` prefixes |
| Commit messages descriptive | MEDIUM 🟡 | Clear what and why, not just what |
| No merge commits in history | LOW 🔵 | Use rebase or squash |
| Related changes in single commit | MEDIUM 🟡 | Don't mix unrelated changes |

## Checklist for Agents

### Before Passing Code Forward

```
Code Quality:
☐ Compiles/runs without errors
☐ Linting passes (ESLint, etc.)
☐ Formatting applied
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
