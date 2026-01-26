---
mode: "agent"
description: "Quick task cycle for small changes (mini architect → dev → review)"
tools: ["codebase", "search", "editFiles", "runCommands", "problems", "testFailure"]
---

# ⚡ Quick Task Cycle

A streamlined development cycle for small, well-defined tasks. Use this for bug fixes, small features, or refactoring tasks that don't need full architecture planning.

## Input Required

```yaml
Task: [What needs to be done]
Type: [bugfix | feature | refactor | chore]
Scope: [Affected area/module]
```

---

## Step 1: 🏗️ Quick Design (2 min)

### Understand
- What exactly needs to change?
- What files are affected?
- What tests need to be written/updated?

### Plan
```markdown
Files to modify:
- [ ] file1.ts - [change description]
- [ ] file2.ts - [change description]

Tests needed:
- [ ] Test case 1
- [ ] Test case 2
```

---

## Step 2: 👨‍💻 TDD Implementation

### Red: Write failing test
```
Create test that defines expected behavior
Run test → Confirm it fails
```

### Green: Make it pass
```
Implement minimal solution
Run test → Confirm it passes
```

### Refactor: Clean up
```
Improve code quality
Run tests → Confirm still green
```

---

## Step 3: 🔍 Self-Review Checklist

Before presenting, verify:

### Code Quality
- [ ] Single responsibility maintained
- [ ] No code duplication introduced
- [ ] Naming is clear and consistent
- [ ] Error handling appropriate

### Testing
- [ ] Tests cover the change
- [ ] Edge cases considered
- [ ] All tests passing

### Security (if applicable)
- [ ] Input validated
- [ ] No sensitive data exposed

---

## 🚨 Validation Gate

**STOP AND PRESENT:**

```markdown
### Task Complete: [Task Name]

**Changes Made:**
- [File]: [What changed]

**Tests Added/Modified:**
- [Test]: [What it verifies]

**Self-Review:**
- Code quality: ✅/⚠️
- Tests passing: ✅/❌
- Security checked: ✅/N/A

**Ready for human review?** Yes/No
```

---

## Quick Commit

```bash
# Bugfix
git commit -m "fix(scope): brief description"

# Feature
git commit -m "feat(scope): brief description"

# Refactor
git commit -m "refactor(scope): brief description"
```
