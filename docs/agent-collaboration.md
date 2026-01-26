# Agent Collaboration Workflow

> This document defines how AI agents help each other improve delivered code through systematic review and refinement cycles.

## Overview

The agent collaboration workflow enables **multi-pass code improvement** where each agent focuses on specific quality dimensions, building on previous agents' work.

## Collaboration Flow

```
┌─────────────────────────────────────────────────────────────┐
│           Agent 1: Implementation                           │
│  - Writes initial code based on requirements               │
│  - Runs basic syntax/lint checks                           │
│  - Self-reviews against conventions                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           Agent 2: Quality Review                           │
│  - Analyzes code structure and readability                 │
│  - Checks against quality standards                        │
│  - Identifies performance/security issues                  │
│  - Suggests architectural improvements                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           Agent 3: Testing & Coverage                       │
│  - Reviews test completeness                               │
│  - Identifies untested edge cases                          │
│  - Suggests additional test scenarios                      │
│  - Validates error handling                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           Agent 4: Documentation & Polish                   │
│  - Verifies documentation completeness                     │
│  - Reviews comments for clarity                            │
│  - Suggests API/UX improvements                            │
│  - Final quality sign-off                                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    DELIVERABLE                              │
│            (Ready for production)                           │
└─────────────────────────────────────────────────────────────┘
```

## Agent Responsibilities

### Agent 1: Implementation Agent

**Focus**: Code creation and basic validation

**Tasks**:
- Write code following project conventions
- Run linting and formatting checks
- Perform basic syntax validation
- Ensure compliance with naming standards
- Self-review against `quality-standards.md`

**Output**: 
- Implementation code
- Self-review notes
- Known limitations/TODOs

**Pass-off Criteria**:
- ✅ Code passes linting
- ✅ Follows naming conventions
- ✅ Compiles/runs without errors
- ✅ Self-review completed

---

### Agent 2: Quality & Architecture Review

**Focus**: Code structure, performance, and security

**Tasks**:
- Analyze code structure against architecture patterns
- Review for performance issues
- Identify security vulnerabilities
- Check for anti-patterns (see `improvement-patterns.md`)
- Validate DRY principle compliance
- Review error handling strategy

**Output**:
- Detailed review feedback
- Specific improvement suggestions
- Risk assessment

**Pass-off Criteria**:
- ✅ No security issues identified
- ✅ Performance acceptable
- ✅ Architectural patterns followed
- ✅ No critical anti-patterns

---

### Agent 3: Testing & Coverage Agent

**Focus**: Test completeness and edge case handling

**Tasks**:
- Review existing tests
- Identify missing test scenarios
- Suggest edge cases to test
- Validate error handling coverage
- Check for integration test gaps
- Propose performance/load tests if relevant

**Output**:
- Test coverage report
- Missing test scenarios
- Test implementation recommendations

**Pass-off Criteria**:
- ✅ Minimum 80% code coverage
- ✅ All critical paths tested
- ✅ Edge cases covered
- ✅ Error scenarios tested

---

### Agent 4: Documentation & Polish Agent

**Focus**: Documentation, API clarity, and final quality

**Tasks**:
- Verify JSDoc/docstring completeness
- Review comment clarity
- Check for dead code
- Validate type definitions
- Suggest API improvements
- Final QA pass

**Output**:
- Documentation enhancements
- Polish recommendations
- Final sign-off

**Pass-off Criteria**:
- ✅ All public APIs documented
- ✅ Complex logic has explanatory comments
- ✅ No dead code
- ✅ Types properly defined
- ✅ Ready for production

---

## Communication Protocol

### Feedback Format

When agents provide feedback, use this structure:

```markdown
## Review: [Focus Area]

### Issue: [Specific Problem]
**Location**: [File/Line reference]
**Severity**: [Critical/High/Medium/Low]
**Description**: [What's wrong and why it matters]

### Recommendation
```typescript
// Suggested fix
```

### Rationale
Explain why this improvement matters and what benefit it provides.

---
```

### Escalation

If an agent encounters issues it can't resolve:

1. **Document the blocker** clearly in feedback
2. **Reference relevant standards** (quality-standards.md, improvement-patterns.md)
3. **Suggest solutions** for the next agent
4. **Mark as requires-decision** if architectural choice needed

## Quality Gates

Each agent must verify these before passing to the next:

```
Agent 1 → Syntax ✓ | Lint ✓ | Conventions ✓
Agent 2 → Architecture ✓ | Performance ✓ | Security ✓
Agent 3 → Coverage ✓ | Edge Cases ✓ | Errors ✓
Agent 4 → Docs ✓ | Types ✓ | Polish ✓
```

## Iteration Cycles

If an agent identifies issues:

1. **Document feedback** with examples
2. **Return to previous agent** or implementation agent
3. **Re-implement** with improvements
4. **Re-review** the same quality gate
5. **Pass forward** once criteria met

## Continuous Improvement

Track improvement patterns across reviews:

- **Common Issues**: Document recurring problems in `improvement-patterns.md`
- **New Standards**: Update `quality-standards.md` when patterns emerge
- **Feedback Loop**: Use agent review comments to improve conventions

## Tools & Integration

Agents should use these resources:

| Resource | Purpose |
|----------|---------|
| `quality-standards.md` | Automated checks to perform |
| `improvement-patterns.md` | Common issues and solutions |
| `agent-review-checklist.md` | Structured review template |
| `conventions.md` | Coding standards |
| `architecture.md` | System design patterns |

## Example Review Interaction

```
Agent 1: "I've implemented getUserById() function - ready for review"

Agent 2: "Found performance issue - N+1 query in user profile loading. 
         Recommend batching queries. See improvement-patterns.md#performance"

Agent 1: "Fixed with batch query optimization"

Agent 3: "Found missing edge case - null profile handling. 
         Suggest test_getUserById_nullProfile scenario"

Agent 1: "Added comprehensive edge case tests"

Agent 4: "Documentation complete - ready for production!"
```
