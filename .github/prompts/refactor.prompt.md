---
mode: "agent"
description: "Safe refactoring workflow with validation at each step"
tools: ["codebase", "search", "editFiles", "runCommands", "problems", "testFailure"]
---

# Refactoring Workflow

Safely refactor code following this structured process.

## Input Required

Provide:
- What code needs refactoring
- Why (code smell, performance, readability, etc.)
- Desired outcome

---

## Phase 1: Assessment

### 1.1 Understand Current State
1. Review the code to be refactored
2. Identify all usages with code search
3. Check existing test coverage
4. Document current behavior

### 1.2 Identify Code Smells
Common issues to look for:
- [ ] Long methods/functions
- [ ] Duplicate code
- [ ] Complex conditionals
- [ ] Poor naming
- [ ] Tight coupling
- [ ] Missing abstraction
- [ ] Dead code

### 1.3 Assess Risk
Evaluate:
- How many files/functions affected?
- How critical is this code path?
- What is the test coverage?
- What could break?

---

## Phase 2: Planning

### 2.1 Define Target State
- What should the code look like after refactoring?
- What patterns should be applied?
- What are the success criteria?

### 2.2 Create Refactoring Plan
Break into small, safe steps:
1. Step 1: [Description]
2. Step 2: [Description]
3. ...

Each step should:
- Be independently testable
- Not break functionality
- Be easy to revert

---

## 🚨 Validation Gate 1: Plan Review

**STOP**: Present the refactoring plan for approval.

Include:
- Current state summary
- Identified issues
- Proposed changes
- Risk assessment
- Step-by-step plan

---

## Phase 3: Preparation

### 3.1 Ensure Test Coverage
Before refactoring:
- Run existing tests - they must pass
- Add tests for any uncovered behavior
- Create a baseline for verification

### 3.2 Document Current Behavior
- Note edge cases
- Document any quirks
- Save current test output

---

## Phase 4: Execute Refactoring

For each step in the plan:

### 4.1 Make Change
- Apply one refactoring step
- Keep changes minimal and focused

### 4.2 Verify
- Run tests
- Check for errors
- Confirm behavior unchanged

### 4.3 Commit Mentally
- If tests pass, proceed to next step
- If tests fail, investigate and fix

---

## Phase 5: Final Validation

### 5.1 Run Full Test Suite
- All tests must pass
- No new lint/type errors
- Performance not degraded

### 5.2 Compare Behavior
- Verify all documented behavior still works
- Check edge cases
- Test integration points

### 5.3 Code Quality Check
Confirm improvements:
- [ ] Code is more readable
- [ ] Duplication removed
- [ ] Better organized
- [ ] Easier to maintain
- [ ] Well documented

---

## 🚨 Validation Gate 2: Completion Review

**STOP**: Present the refactoring results for review.

Summary should include:
1. What was refactored
2. Improvements achieved
3. All tests passing
4. Any concerns or follow-up needed
