# Feature Specification: [Feature Name]

> Template for creating implementation-ready specifications.
> Delete this quote block and fill in the sections below.

## Overview

### Problem Statement
What problem does this feature solve? Why is it needed?

### Proposed Solution
High-level description of the solution.

### Success Criteria
How do we know this feature is complete and working?
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

---

## Requirements

### Functional Requirements
What must the feature do?

1. **FR-1**: [Description]
2. **FR-2**: [Description]
3. **FR-3**: [Description]

### Non-Functional Requirements
Performance, security, accessibility, etc.

1. **NFR-1**: [Description]
2. **NFR-2**: [Description]

### Out of Scope
What is explicitly NOT included in this feature?

- Item 1
- Item 2

---

## Technical Design

### Architecture
How does this fit into the existing system?

```
[Diagram or description of component interactions]
```

### Data Model
New or modified data structures:

```typescript
interface Example {
  id: string;
  // ... fields
}
```

### API Design
New or modified endpoints:

```
POST /api/v1/resource
Request: { ... }
Response: { ... }
```

### Dependencies
External libraries, services, or internal modules needed:

- Dependency 1: [Purpose]
- Dependency 2: [Purpose]

---

## Implementation Plan

### Tasks
Break down into implementable tasks:

1. [ ] **Task 1**: [Description] - Est: [X hours]
2. [ ] **Task 2**: [Description] - Est: [X hours]
3. [ ] **Task 3**: [Description] - Est: [X hours]

### Testing Strategy
How will this be tested?

- **Unit Tests**: [What to test]
- **Integration Tests**: [What to test]
- **Manual Testing**: [Scenarios to verify]

### Rollout Plan
How will this be deployed?

1. Step 1
2. Step 2
3. Step 3

---

## Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Risk 1 | High/Med/Low | High/Med/Low | Mitigation strategy |
| Risk 2 | High/Med/Low | High/Med/Low | Mitigation strategy |

---

## Open Questions

- [ ] Question 1?
- [ ] Question 2?

---

## References

- [Link to related docs]
- [Link to design files]
- [Link to related tickets]

---

**Author**: [Name]  
**Created**: [Date]  
**Last Updated**: [Date]  
**Status**: Draft | In Review | Approved | Implemented
