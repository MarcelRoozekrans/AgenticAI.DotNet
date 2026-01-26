# Agent Review Checklist

> A structured checklist for agents to use during code review. Copy and adapt based on which quality dimension you're focusing on.

## How to Use This Checklist

1. **Select your focus area** based on which agent you are (1-4)
2. **Work through each section** systematically
3. **Document findings** with specific file references and line numbers
4. **Use improvement-patterns.md** to suggest fixes
5. **Reference quality-standards.md** for standards
6. **Provide actionable feedback** with examples

---

## Agent 1: Implementation Review

**Focus**: Basic code quality, syntax, conventions, initial validation

### Syntax & Compilation

- [ ] Code compiles without errors
- [ ] No TypeScript errors or unresolved types
- [ ] All imports resolve correctly
- [ ] No console errors when running

### Code Style

- [ ] File names follow kebab-case convention
- [ ] Class names use PascalCase
- [ ] Function names use camelCase
- [ ] Constants use UPPER_SNAKE_CASE
- [ ] Variable names are descriptive (not `x`, `temp`, `data`)
- [ ] Indentation consistent (2 spaces as per project)
- [ ] Lines under 100 characters (or justified)
- [ ] No trailing whitespace

### Linting & Formatting

- [ ] ESLint passes: `npm run lint`
- [ ] Prettier applied: `npm run format`
- [ ] No eslint-disable comments without justification
- [ ] No unused imports
- [ ] No unused variables

### Basic Structure

- [ ] Functions under 30 lines (or justified)
- [ ] Maximum 3-4 parameters per function (or uses options object)
- [ ] No code duplication visible
- [ ] Clear code organization within file

### Type Safety

- [ ] All function parameters have types
- [ ] All function return types specified
- [ ] No `any` types without JSDoc comment explaining why
- [ ] Type definitions for complex objects

### Documentation

- [ ] Public functions have JSDoc comments
  ```typescript
  /**
   * Brief description
   * @param name - Description
   * @returns Description
   */
  ```
- [ ] Complex logic has explanatory comments
- [ ] Comment explains "why", not "what"
- [ ] No stale comments that contradict code

### Basic Error Handling

- [ ] Try-catch blocks present for operations that can fail
- [ ] Errors logged or handled explicitly
- [ ] No silent failures (catch without re-throw)
- [ ] Error types are specific (not generic `Error`)

### Self-Review Assessment

**Code Quality Score**: ___/10

**Issues Found**: 
- [ ] None - Ready for Quality Review
- [ ] Minor - Document and continue
- [ ] Major - Requires refactoring

---

## Agent 2: Quality & Architecture Review

**Focus**: Performance, security, architecture patterns, design quality

### Architecture & Design Patterns

- [ ] Follows project architecture patterns
- [ ] Appropriate design patterns used
- [ ] Single Responsibility Principle followed
- [ ] Dependencies properly organized
- [ ] No circular dependencies
- [ ] Proper separation of concerns (UI/business/data)

### Performance Issues

- [ ] No N+1 query patterns (see `improvement-patterns.md`)
- [ ] No unnecessary re-renders (React components)
- [ ] Batch operations for related queries
- [ ] No synchronous blocking operations
- [ ] Appropriate data structures (Set vs Array, Map for lookups)
- [ ] No obvious memory leaks
- [ ] Caching used appropriately
- [ ] Database indexes assumed for queries

**Performance Concerns**:
- [ ] None found
- [ ] Minor optimization suggestions
- [ ] Significant performance issues identified

### Security Review

- [ ] No hardcoded secrets or API keys
- [ ] All secrets use environment variables
- [ ] Input validation on all user-facing functions
- [ ] No SQL injection vulnerabilities
- [ ] Parameterized queries used consistently
- [ ] No XSS vulnerabilities (web)
- [ ] Authentication/authorization checked
- [ ] No sensitive data in logs
- [ ] Dependencies checked with `npm audit`
- [ ] No known CVEs in dependencies

**Security Concerns**:
- [ ] None found
- [ ] Minor suggestions
- [ ] Critical vulnerabilities identified

### Code Complexity

- [ ] Cyclomatic complexity < 10 (functions)
- [ ] No deep nesting (< 4 levels)
- [ ] Logic easy to follow
- [ ] Complex algorithms explained in comments
- [ ] No "magic numbers" without constants

**Refactoring Suggestions**:
- [ ] None needed
- [ ] Some simplifications possible
- [ ] Significant refactoring needed

### Error Handling Strategy

- [ ] Specific error types defined
- [ ] Errors thrown with descriptive messages
- [ ] Errors propagate appropriately
- [ ] Try-catch at correct levels
- [ ] Graceful degradation where appropriate
- [ ] Error context preserved (not lost in re-throws)

### Resource Management

- [ ] Timers properly cleaned up
- [ ] Event listeners removed
- [ ] Streams properly closed
- [ ] Database connections released
- [ ] No resource leaks

### Maintainability

- [ ] Code is self-documenting
- [ ] Naming is clear and semantic
- [ ] Complex logic decomposed into functions
- [ ] Similar patterns used consistently
- [ ] No god objects or overly complex classes

### Anti-patterns Check

Use `improvement-patterns.md` to verify:

- [ ] No N+1 queries
- [ ] No callback hell
- [ ] No unnecessary re-renders
- [ ] No god objects
- [ ] No memory leaks
- [ ] No hardcoded secrets
- [ ] No missing input validation
- [ ] Errors properly handled
- [ ] No magic numbers
- [ ] No code duplication

**Quality Score**: ___/10

**Improvement Level**: 
- [ ] Excellent - Ready for testing
- [ ] Good - Minor improvements suggested
- [ ] Needs Work - Significant refactoring needed

---

## Agent 3: Testing & Coverage Review

**Focus**: Test completeness, edge cases, error scenarios, coverage

### Coverage Assessment

- [ ] Overall code coverage >= 80%
- [ ] Coverage tool reports reviewed
  ```
  ☐ Statements >= 80%
  ☐ Branches >= 75%
  ☐ Functions >= 80%
  ☐ Lines >= 80%
  ```
- [ ] Critical paths 100% covered
- [ ] Low coverage areas identified and justified

### Test Organization

- [ ] Tests co-located with source or in `__tests__`
- [ ] Test file naming consistent
- [ ] Test names describe behavior
- [ ] Tests organized in describe blocks
- [ ] Clear setup/teardown
- [ ] No test interdependencies

**Test Naming Convention Check**:
```
☐ test_functionName_expectedBehavior
☐ test_functionName_givenCondition_expectedResult
Example: test_getUserById_returnsUserWhenFound
```

### Happy Path Tests

- [ ] Main functionality tested
- [ ] Expected inputs and outputs verified
- [ ] Return values validated
- [ ] Side effects verified (DB writes, etc.)

### Edge Case Tests

- [ ] null/undefined inputs tested
- [ ] Empty array/collection tested
- [ ] Boundary values tested
- [ ] Zero/negative values tested (if applicable)
- [ ] Maximum values tested
- [ ] Special characters tested (if string)
- [ ] Whitespace handling tested (if string)

### Error Scenario Tests

- [ ] Each `throw` statement has a test
- [ ] Error types verified
- [ ] Error messages checked
- [ ] Error conditions clearly specified
- [ ] Recovery paths tested

### Async/Promise Tests

- [ ] Async functions properly awaited
- [ ] Promise chains tested
- [ ] Rejection scenarios tested
- [ ] Timeout handling tested
- [ ] No race conditions

### Integration Tests

- [ ] API endpoints have integration tests
- [ ] Database interactions tested
- [ ] External service calls tested (mocked)
- [ ] Multi-step workflows tested
- [ ] State transitions validated

### Test Quality

- [ ] Tests are deterministic (not flaky)
- [ ] No sleep/setTimeout in tests (use fake timers)
- [ ] Proper test isolation (no shared state)
- [ ] Mocks/stubs appropriate and documented
- [ ] Test data is realistic
- [ ] No `test.skip` without issue reference
- [ ] No commented-out tests

### Coverage Gaps

**Missing Test Scenarios**:
- [ ] [List specific scenarios not covered]

**Recommendations**:
- [ ] [Suggest new tests to add]

### Test Execution

- [ ] All tests pass locally
- [ ] Tests run in CI/CD
- [ ] Test suite runs in reasonable time (< 60 seconds)
- [ ] No flaky tests

**Test Results**:
- [ ] ✅ All pass - Ready for deployment
- [ ] ⚠️ Some failures - Document before passing
- [ ] ❌ Critical failures - Cannot pass forward

### Coverage Verdict

**Overall Test Assessment**: ___/10

**Readiness**:
- [ ] Excellent - Well tested, ready to deploy
- [ ] Good - Coverage adequate, minor gaps
- [ ] Needs Improvement - Add more tests before deployment
- [ ] Inadequate - Significant gaps, requires more testing

---

## Agent 4: Documentation & Polish Review

**Focus**: Documentation completeness, API clarity, type definitions, final QA

### API Documentation

- [ ] All public functions have JSDoc
- [ ] JSDoc includes description, params, returns, throws
- [ ] Complex types documented
- [ ] Usage examples provided for complex functions
- [ ] Parameter types clearly specified
- [ ] Return types clearly specified
- [ ] Exceptions documented (@throws)

**JSDoc Template Check**:
```
☐ One-line description
☐ Longer description if needed
☐ @param tags with type and description
☐ @returns tag with type and description
☐ @throws tags for thrown errors
☐ @example with realistic usage
```

### Code Comments

- [ ] Complex logic has explanatory comments
- [ ] Comments explain "why", not "what"
- [ ] No obvious/redundant comments
- [ ] TODO comments have context or issue reference
- [ ] No stale/outdated comments

### Type Definitions

- [ ] All public functions typed
- [ ] Complex objects have type definitions
- [ ] Interface names descriptive
- [ ] Generic types properly constrained
- [ ] No `any` without justification
- [ ] Optional fields clearly marked

### README & User-Facing Docs

- [ ] New features documented in README
- [ ] Setup instructions up-to-date
- [ ] API changes documented
- [ ] Breaking changes clearly marked
- [ ] Examples accurate and runnable

### Code Cleanliness

- [ ] No dead code (unused functions/variables)
- [ ] No commented-out code blocks
- [ ] No debug statements left in
- [ ] No console.log in production code
- [ ] Imports organized and cleaned up
- [ ] No circular dependencies

### Type Completeness

- [ ] Function parameters typed
- [ ] Return types specified
- [ ] Callbacks typed
- [ ] Generics properly constrained
- [ ] Type exports available if public

### Error Messages

- [ ] Error messages are descriptive
- [ ] Error messages include context
- [ ] No cryptic or generic messages
- [ ] User-facing errors are friendly
- [ ] Developer errors are technical

### Constants & Configuration

- [ ] Magic numbers extracted to constants
- [ ] Configuration centralized
- [ ] Environment variables documented
- [ ] Defaults reasonable

### Accessibility (Web)

- [ ] ARIA labels present
- [ ] Semantic HTML used
- [ ] Color contrast adequate
- [ ] Keyboard navigation works
- [ ] Alt text for images

### Final QA Checks

**Deployment Readiness**:
- [ ] Code review complete
- [ ] Tests passing
- [ ] Coverage adequate
- [ ] Documentation complete
- [ ] No breaking changes undocumented
- [ ] Performance acceptable
- [ ] Security reviewed
- [ ] Dependencies audited

### Sign-Off Assessment

**Documentation Quality**: ___/10

**Overall Readiness**:
- [ ] ✅ **APPROVED** - Ready for production
- [ ] ⚠️ **CONDITIONAL** - Ready with documented assumptions
- [ ] ❌ **BLOCKED** - Requires changes before deployment

**Final Notes**:
- [ ] All critical items addressed
- [ ] No known issues or workarounds
- [ ] Ready for release

---

## Feedback Template

Use this when providing review feedback:

```markdown
## Review Summary

**Focus Area**: [Agent 1/2/3/4 - What you reviewed]  
**Overall Status**: [✅ Pass / ⚠️ Minor Issues / ❌ Needs Work]

## Key Findings

### ✅ Strengths
- [What was done well]

### ⚠️ Issues Found

#### Issue 1: [Title]
**Location**: [file.ts:45](file.ts#L45)  
**Severity**: [Critical/High/Medium/Low]  
**Details**: [What's wrong]  

**Recommendation**:
```typescript
// Suggested fix
```

**Reference**: See `quality-standards.md#section` and `improvement-patterns.md#pattern`

---

#### Issue 2: [Title]
[Same structure]

### 📋 Checklist Status

| Item | Status |
|------|--------|
| Syntax/Compilation | ✅/⚠️/❌ |
| Type Safety | ✅/⚠️/❌ |
| Documentation | ✅/⚠️/❌ |
| Error Handling | ✅/⚠️/❌ |
| Performance | ✅/⚠️/❌ |
| Security | ✅/⚠️/❌ |
| Tests | ✅/⚠️/❌ |

## Verdict

**Pass to Next Agent**: ✅ Yes / ❌ No  
**Required Actions**: 
- [ ] [Action 1]
- [ ] [Action 2]

**Next Agent**: [Agent 1/2/3/4]
```

---

## Tips for Effective Reviews

1. **Be Specific**: Reference exact files and line numbers
2. **Show Examples**: Provide code snippets showing the issue and solution
3. **Explain Why**: Help the original author understand the reasoning
4. **Be Constructive**: Focus on improving the code, not criticizing
5. **Reference Standards**: Link to quality-standards.md and improvement-patterns.md
6. **Suggest Solutions**: Don't just point out problems
7. **Acknowledge Wins**: Highlight what was done well
8. **Keep Records**: Document patterns for future reference
