---
mode: "agent"
description: "Structured bug fixing workflow with root cause analysis"
tools: ["codebase", "search", "editFiles", "runCommands", "problems", "testFailure"]
---

# Bug Fix Workflow

Fix a bug following this structured debugging process.

## Input Required

Provide:
- Bug description or error message
- Steps to reproduce (if known)
- Expected vs actual behavior

---

## Phase 1: Investigation

### 1.1 Understand the Bug
- What is the reported behavior?
- What is the expected behavior?
- When did it start happening?
- Is it reproducible?

### 1.2 Gather Evidence
1. Search for related error messages in the codebase
2. Check recent changes to affected files
3. Review any error logs or stack traces
4. Use the problems tool to find related issues

### 1.3 Reproduce the Issue
- Identify the exact steps to trigger the bug
- Note any specific conditions required
- Document the error output

---

## Phase 2: Root Cause Analysis

### 2.1 Trace the Code Path
1. Start from the error location
2. Follow the execution flow backwards
3. Identify where the bug originates

### 2.2 Identify Root Cause
Document:
- What is causing the bug?
- Why was this not caught before?
- Are there similar patterns elsewhere?

### 2.3 Consider Solutions
Think of at least 2 potential fixes:
1. **Solution A**: [Description, pros, cons]
2. **Solution B**: [Description, pros, cons]

---

## 🚨 Validation Gate 1: Diagnosis Review

**STOP**: Present your root cause analysis and proposed solutions for approval.

Include:
- Root cause explanation
- Proposed solutions with trade-offs
- Recommendation and rationale

---

## Phase 3: Fix Implementation

### 3.1 Apply the Fix
- Make the minimal necessary changes
- Follow project conventions
- Add defensive code where appropriate

### 3.2 Add Regression Test
Write a test that:
- Reproduces the original bug
- Verifies the fix works
- Prevents future regression

### 3.3 Check for Similar Issues
- Search for similar patterns in the codebase
- Fix related occurrences if found
- Note any that need separate attention

---

## Phase 4: Verification

### 4.1 Run Tests
- Execute the new regression test
- Run the full test suite
- Check for lint/type errors

### 4.2 Manual Verification
- Reproduce the original bug steps
- Confirm the bug is fixed
- Check for side effects

---

## 🚨 Validation Gate 2: Fix Review

**STOP**: Present the completed fix for review.

Summary should include:
1. Root cause (brief)
2. Fix applied
3. Regression test added
4. Any related issues found
5. Verification results
