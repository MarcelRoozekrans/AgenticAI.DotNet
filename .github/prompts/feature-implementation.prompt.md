---
mode: "agent"
description: "Feature implementation workflow from specification to code"
tools: ["codebase", "search", "editFiles", "runCommands", "problems", "testFailure"]
---

# Feature Implementation Workflow

Implement a new feature following this structured process.

## Input Required

Provide:
- Feature specification or description
- Related files or existing patterns to follow

---

## Phase 1: Understanding & Planning

### 1.1 Context Loading
1. Review the [project architecture](../../docs/architecture.md)
2. Check relevant [instruction files](../../.github/instructions/)
3. Search for similar implementations in the codebase

### 1.2 Requirements Clarification
- What is the expected behavior?
- What are the inputs and outputs?
- Are there any constraints or edge cases?
- How does this integrate with existing code?

### 1.3 Implementation Plan
Create a brief plan:
1. Files to create or modify
2. Dependencies needed
3. Testing approach

---

## 🚨 Validation Gate 1: Plan Review

**STOP**: Present the implementation plan for approval before writing code.

---

## Phase 2: Implementation

### 2.1 Write Code
Follow the project conventions and instruction files:
- Use existing patterns from the codebase
- Follow naming conventions
- Add appropriate error handling
- Include inline documentation

### 2.2 Add Tests
Write tests alongside the implementation:
- Unit tests for individual functions
- Integration tests for API endpoints
- Edge cases and error scenarios

### 2.3 Update Documentation
- Add JSDoc/docstrings to new functions
- Update relevant documentation files
- Add usage examples if applicable

---

## Phase 3: Validation

### 3.1 Run Checks
- Run linter and fix any issues
- Run type checker
- Execute test suite
- Check for problems in the IDE

### 3.2 Self-Review
Verify:
- [ ] Code follows project conventions
- [ ] All acceptance criteria are met
- [ ] Tests are passing
- [ ] No lint/type errors
- [ ] Documentation is updated

---

## 🚨 Validation Gate 2: Implementation Review

**STOP**: Present the completed implementation for review before considering it done.

Summary should include:
1. What was implemented
2. Files created/modified
3. Tests added
4. Any concerns or trade-offs made
