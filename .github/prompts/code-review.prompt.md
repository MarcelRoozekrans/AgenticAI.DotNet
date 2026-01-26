---
mode: "agent"
description: "Comprehensive code review workflow with validation gates"
tools: ["codebase", "search", "problems", "testFailure"]
---

# Code Review Workflow

Perform a comprehensive code review following this structured process.

## Phase 1: Context Loading

1. Review the [project conventions](../../docs/conventions.md)
2. Check relevant [instruction files](../../.github/instructions/) for the code domain
3. Understand the purpose of the changes being reviewed

## Phase 2: Static Analysis

Use semantic search to understand the changed code:
- Identify the files and functions modified
- Check for any existing lint or type errors using the problems tool
- Review test results with testFailure if tests were run

## Phase 3: Code Quality Review

Evaluate each file for:

### Correctness
- [ ] Logic is correct and handles edge cases
- [ ] Error handling is appropriate
- [ ] No obvious bugs or issues

### Readability
- [ ] Code is easy to understand
- [ ] Naming is clear and consistent
- [ ] Comments explain "why" not "what"
- [ ] Functions are focused and not too long

### Maintainability
- [ ] DRY principle followed
- [ ] Appropriate abstraction level
- [ ] No unnecessary complexity
- [ ] Easy to modify in the future

## Phase 4: Security Review

Check for:
- [ ] Input validation on all user inputs
- [ ] No hardcoded secrets or credentials
- [ ] Proper authentication/authorization
- [ ] SQL injection prevention
- [ ] XSS prevention (frontend)

## Phase 5: Performance Review

Consider:
- [ ] No N+1 query problems
- [ ] Appropriate caching
- [ ] No unnecessary computations
- [ ] Efficient algorithms

## Phase 6: Testing Review

Verify:
- [ ] Tests exist for new functionality
- [ ] Edge cases are covered
- [ ] Tests are meaningful (not just for coverage)
- [ ] Mocking is appropriate

---

## 🚨 Human Validation Gate

**STOP**: Present your review findings before providing the final summary.

Organize findings by severity:
1. **🔴 Critical** - Must fix before merge
2. **🟠 Major** - Should fix before merge  
3. **🟡 Minor** - Consider fixing
4. **🔵 Suggestion** - Nice to have improvements

Include positive feedback on what was done well.
