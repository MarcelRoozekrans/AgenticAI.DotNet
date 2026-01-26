# Inner Loop vs. Outer Loop: Two Modes of Development

This document explains how to use GitHub's agentic primitives framework in two distinct modes: **Inner Loop** (focused, single-session development) and **Outer Loop** (orchestrated, team-coordinated workflow).

> Reference: [GitHub Blog - Agentic Primitives](https://github.blog/ai-and-ml/github-copilot/how-to-build-reliable-ai-workflows-with-agentic-primitives-and-context-engineering/)

---

## 🔄 What's the Difference?

| Aspect | Inner Loop | Outer Loop |
|--------|-----------|-----------|
| **Participants** | Single developer | Team (developer, reviewer, architect) |
| **Duration** | 20-30 minutes | Multiple sessions over hours/days |
| **Sessions** | ONE continuous session | SEPARATE sessions per phase |
| **Context** | Shared within session | Managed via `.memory.md` and `.spec.md` |
| **Decision Gate** | Real-time developer intuition | Explicit human approval at gates |
| **Best For** | Bug fixes, small features, exploration | Complex features, team coordination, production |
| **Risk Level** | Lower (isolated changes) | Higher (affects multiple components) |
| **AI Model** | Can be more experimental | Must be systematic and traceable |

---

## 🎯 Inner Loop: Single Developer, Single Session

### When to Use Inner Loop

✅ **Good for:**
- Bug fixes and hotfixes
- Small feature additions
- Code refactoring and cleanup
- Experimental changes
- Prototyping and exploration
- Learning and investigation

❌ **Not ideal for:**
- Complex features (> 2-3 hours work)
- Features affecting multiple domains
- Security-critical changes
- Production deployments
- Team hand-offs

### Inner Loop Workflow

```
Developer Start Session
        ↓
[In VS Code: @dev]
        ↓
┌─────────────────────────────────────────┐
│ Phase: IMPLEMENTATION (Single Session)  │
├─────────────────────────────────────────┤
│ 1. Design component/fix                 │
│ 2. Write tests (Red)                    │
│ 3. Implement (Green)                    │
│ 4. Refactor (Blue)                      │
│ 5. Run static analysis                  │
│ 6. Commit to branch                     │
└─────────────────────────────────────────┘
        ↓
Continuous local iteration
(Test → Code → Test)
        ↓
Push branch when satisfied
(Commit message: task summary)
        ↓
✅ Ready for review in outer loop
```

### Inner Loop Example: Fix a Bug

```markdown
# In VS Code Chat
@dev fix: User email validation not working for subdomains

## What I Did
- Wrote failing test for subdomain emails ✓
- Fixed regex in email validator ✓
- Refactored to use built-in email validation ✓
- All tests passing ✓
- Committed: fix: email validation for subdomains

## Changes
- `src/validators/email.ts` - Updated regex
- `tests/validators/email.spec.ts` - Added subdomain test

Ready for @reviewer when you are!
```

### Inner Loop Tools & Context

- **Chatmode**: `@dev` (developer mode)
- **Tools**: Full access (codebase, files, commands, test runner)
- **Memory**: Current session state only
- **Validation**: Automated (tests, linting)
- **Decision Approval**: Developer's judgment

---

## 🚀 Outer Loop: Team-Coordinated Workflow

### When to Use Outer Loop

✅ **Good for:**
- Complex features (multiple hours of work)
- Features spanning multiple components
- Security-critical or high-risk changes
- Team learning and knowledge transfer
- Production deployments
- Features requiring architecture review
- Major refactors

❌ **Not ideal for:**
- Simple one-file fixes
- Typo corrections
- Very small improvements
- When speed is critical and risk is low

### Outer Loop Workflow (5 Phases)

```
User: "Let's build a new user authentication system"
        ↓
┌─────────────────────────────────────────────────────┐
│ Phase 1: ARCHITECT (30-45 min, separate session)    │
│ Chatmode: @architect (read-only, planning only)     │
├─────────────────────────────────────────────────────┤
│ 1. Understand requirements                          │
│ 2. Design architecture                              │
│ 3. Create `docs/auth-system.spec.md`                │
│ 4. Break down into tasks                            │
│ 5. Document risks & mitigations                     │
│                                                     │
│ Gate 1: User reviews & approves spec                │
└─────────────────────────────────────────────────────┘
        ↓ [NEW SESSION - Fresh context]
┌─────────────────────────────────────────────────────┐
│ Phase 2: DEVELOPER (60-90 min, separate session)    │
│ Chatmode: @developer (full access, implementation)  │
├─────────────────────────────────────────────────────┤
│ 1. Load spec from Phase 1                           │
│ 2. Implement each task:                             │
│    - Write failing tests (Red)                      │
│    - Implement code (Green)                         │
│    - Refactor (Blue)                                │
│ 3. All tests passing                                │
│ 4. Commit with descriptive messages                 │
│                                                     │
│ Gate 2: Developer confirms completion               │
└─────────────────────────────────────────────────────┘
        ↓ [NEW SESSION - Fresh context]
┌─────────────────────────────────────────────────────┐
│ Phase 3: REVIEWER (30-45 min, separate session)     │
│ Chatmode: @reviewer (read-only, QA focus)           │
├─────────────────────────────────────────────────────┤
│ 1. Load spec and implementation                     │
│ 2. Check against acceptance criteria                │
│ 3. Security review                                  │
│ 4. SOLID principles compliance                      │
│ 5. Create findings document                         │
│ 6. Approve or request changes                       │
│                                                     │
│ Gate 3: Approve or → Phase 4                        │
└─────────────────────────────────────────────────────┘
        ↓ [If Issues Found]
┌─────────────────────────────────────────────────────┐
│ Phase 4: ITERATE (60-90 min, repeat as needed)      │
│ Back to @developer to fix issues                    │
│                                                     │
│ Then → Phase 3 again (New Review Session)           │
└─────────────────────────────────────────────────────┘
        ↓ [When Approved]
┌─────────────────────────────────────────────────────┐
│ Phase 5: VERIFY & COMPLETE (30-45 min)              │
│ Chatmode: @developer (verification focus)           │
├─────────────────────────────────────────────────────┤
│ 1. Static analysis: PASS ✓                          │
│ 2. All tests: PASS ✓                                │
│ 3. Build: PASS ✓                                    │
│ 4. Runtime verification: PASS ✓                     │
│ 5. Create summary document                          │
│ 6. Merge to main branch                             │
│                                                     │
│ Gate 4: Production ready ✓                          │
└─────────────────────────────────────────────────────┘
        ↓
✅ FEATURE COMPLETE & DEPLOYED
```

### Outer Loop Key Principles

1. **Separate Sessions Per Phase**
   - Fresh context prevents hallucination/drift
   - Each role has focused responsibility
   - Clear handoff points

2. **Spec-Driven Development**
   - Phase 1 creates `*.spec.md`
   - Phase 2 implements from spec (not AI-designed changes)
   - Ensures traceability and determinism

3. **Memory-Driven Continuity**
   - `.memory.md` preserves patterns between sessions
   - Prevents repeating mistakes
   - Documents learnings as you go

4. **Validation Gates**
   - Gate 1: Architecture approved before implementation
   - Gate 2: Implementation complete before review
   - Gate 3: Issues resolved before verification
   - Gate 4: All checks passing before deployment

5. **Role-Based Tool Boundaries**
   - Architect: Read-only, planning only
   - Developer: Full access, implementation
   - Reviewer: Read-only, quality checks
   - Prevents accidental or malicious changes

### Outer Loop Example: New User Authentication Feature

**Starting prompt** (user provides to @architect):

```markdown
# Build: OAuth 2.0 Integration with GitHub

We need users to log in with their GitHub account instead of username/password.

Requirements:
- Support GitHub OAuth 2.0 flow
- Create user on first login
- Link existing users by email
- Store OAuth token securely
- Expire sessions after 7 days

Due: Friday EOD
Risk: Medium (affects authentication)
```

**Phase 1 Deliverable** (architect creates `.github/oauth-integration.spec.md`):

```markdown
# Feature Specification: OAuth 2.0 GitHub Integration

## Problem Statement
Current username/password system has weak security and poor UX.

## Requirements
[Detailed breakdown from user input]

## Technical Design
```
GitHub OAuth Flow
    ↓
Auth Service receives code
    ↓
Exchange for token
    ↓
Fetch user info
    ↓
Create or update User record
    ↓
Issue session cookie
```

## Task Breakdown
- [ ] Task 1: Create OAuth routes (~2h)
- [ ] Task 2: Implement token exchange (~2h)
- [ ] Task 3: Integrate GitHub API (~2h)
- [ ] Task 4: Add session management (~2h)
- [ ] Task 5: Add security tests (~2h)

## Acceptance Criteria
- [ ] User can log in with GitHub
- [ ] Account auto-created on first login
- [ ] Email-based linking for existing users
- [ ] OAuth token stored encrypted
- [ ] All security tests passing
```

**Gate 1**: User reviews spec, asks questions, approves.

**Phase 2**: Developer implements all 5 tasks in same session (using spec as blueprint).

**Phase 3**: Reviewer checks against spec, runs security review, finds 2 issues.

**Gate 3**: Issues found, need Phase 4.

**Phase 4**: Developer fixes 2 issues, commits improvements.

**Phase 3 (Re-run)**: Reviewer approves all changes.

**Phase 5**: Final verification, all checks pass, merged to main.

---

## 📋 How to Choose: Inner or Outer Loop?

### Decision Tree

```
Is the change complex
or requires multiple domains?
├─ YES → Use OUTER LOOP
│   (Team coordination, explicit gates)
│
└─ NO → Is it security-critical?
    ├─ YES → Use OUTER LOOP
    │   (Security review needed)
    │
    └─ NO → Will it affect multiple developers?
        ├─ YES → Use OUTER LOOP
        │   (Knowledge transfer value)
        │
        └─ NO → INNER LOOP is fine
            (Fast, focused, isolated)
```

### Quick Reference

**Use Inner Loop** for:
- `git commit -m "fix: typo in error message"`
- `git commit -m "chore: update dependencies"`
- `git commit -m "refactor: simplify validation logic"`
- `git commit -m "test: add edge case coverage"`

**Use Outer Loop** for:
- `git commit -m "feat: new OAuth authentication system"`
- `git commit -m "feat: migrate to microservices architecture"`
- `git commit -m "security: implement rate limiting"`
- `git commit -m "perf: rewrite database query engine"`

---

## 🔧 Configuration

Both modes are configured in `.github/apm.yml`:

```yaml
inner-loop:
  description: "Developer mode - full access within session"
  session_mode: "streaming"
  iteration_time: "20-30 minutes"
  scope: "Single component/feature"

outer-loop:
  description: "Team mode - coordinated phases with handoffs"
  session_mode: "discrete_phases"
  phase_duration: "30-45 minutes each"
  phases: ["Architect", "Developer", "Reviewer", "Iterate", "Verify"]
```

---

## 📚 Related Documentation

- [Development Cycle Prompt](.github/prompts/development-cycle.prompt.md) - Outer Loop orchestration
- [Developer Chatmode](.github/chatmodes/developer.chatmode.md) - Inner Loop focused work
- [Project Memory](.memory.md) - Shared knowledge across sessions
- [Specification Template](docs/specs/_template.spec.md) - Phase 1 output format

---

## 💡 Best Practices

1. **Start Simple** - Use inner loop for your first few changes
2. **Graduate to Outer** - Move to outer loop as complexity grows
3. **Update Memory** - Record patterns and learnings as you go
4. **Respect Gates** - Don't skip validation checkpoints
5. **Session Hygiene** - Always start fresh sessions between phases
6. **Document Decisions** - Add rationale to `.memory.md`

