# MCP Tool Boundaries: Security & Safety Guidelines

This document defines which Model Context Protocol (MCP) tools are available to each role in the agentic development workflow, ensuring security, safety, and appropriate access control.

> **MCP (Model Context Protocol)**: Standardized protocol for AI models to safely access external systems through defined interfaces.

---

## 🔐 Security Model

### Principle: Role-Based Access Control (RBAC)

Each role gets exactly the tools it needs, nothing more:

- **Architect**: Planning tools (read-only)
- **Developer**: Implementation tools (read + write + execute)
- **Reviewer**: Quality tools (read-only)
- **User**: Human decision maker (approval authority)

---

## 🛠️ Tool Inventory

### ✅ Safe Tools (Available to Multiple Roles)

| Tool | Purpose | Risk Level | Who Uses |
|------|---------|-----------|----------|
| **Codebase Search** | Find files, grep, semantic search | Low | All roles |
| **Problems** | View linting/compilation errors | Low | All roles |
| **Documentation** | Read docs, specs, comments | Low | All roles |
| **Memory** | Read `.memory.md` | Low | All roles |

### 📝 Write Tools (Limited to Developer Role)

| Tool | Purpose | Risk Level | Who Uses | Constraints |
|------|---------|-----------|----------|-------------|
| **Edit Files** | Modify code files | Medium | Developer | Only staged files from spec |
| **Create Files** | Add new files | Medium | Developer | Must follow naming conventions |
| **Delete Files** | Remove files | High | Developer | ONLY within Phase 4 (iterate) |

### 🚀 Execution Tools (Limited to Developer Role)

| Tool | Purpose | Risk Level | Who Uses | Constraints |
|------|---------|-----------|----------|-------------|
| **Run Commands** | Execute scripts/tests | High | Developer | Sandboxed, whitelisted commands |
| **Test Runner** | Run test suite | Medium | Developer | Read-only database, no data destroy |
| **Build** | Compile/transpile code | Medium | Developer | Sandbox environment only |

### ⚠️ Restricted Tools (Not Available to AI)

| Tool | Reason | Human Alternative |
|------|--------|-------------------|
| **Network Access** | Could exfiltrate data | Manual API integration |
| **Database Direct Access** | Could modify/delete data | Migrations + seeding scripts |
| **Production Deployment** | Could affect live users | GitHub Actions with approval |
| **Secret Management** | Could expose credentials | Manual `.env.example` setup |
| **User Account Access** | Privacy violation | Admin oversight only |
| **System Admin** | Complete control risk | Manual infrastructure changes |

---

## 👨‍🏗️ Architect Role: Planning Mode

### Available Tools

```yaml
role: "architect"
mode: "read-only"
session_duration: "30-45 minutes"
tools:
  - codebase:
      operations: ["search", "read", "grep"]
      write_access: false
  - documentation:
      can_read: true
      can_write: false
  - memory:
      can_read: true
      can_update: false
  - problems:
      can_view: true
      can_dismiss: false
```

### What Architect Can Do ✅

- Read existing code and architecture
- Search for patterns and precedents
- View error/lint reports
- Access documentation
- Read project memory
- **ONLY OUTPUT**: Markdown specifications, diagrams, task breakdowns
- **NO EXECUTION**: Cannot run any code

### What Architect Cannot Do ❌

- Modify any files
- Run commands or tests
- Access external APIs
- Make decisions without user approval
- Execute any changes

### Architect Workflow

```
@architect: "Design the OAuth system"
       ↓
Read current auth system ✓
Read authentication patterns ✓
Read security policies ✓
View project conventions ✓
       ↓
Generate markdown spec ✓
       ↓
Present to user for approval
       ↓
User: "Looks good!" ✓ or "Change X, Y, Z" ✓
```

---

## 👨‍💻 Developer Role: Implementation Mode

### Available Tools

```yaml
role: "developer"
mode: "read-write-execute"
session_duration: "60-90 minutes"
tools:
  - codebase:
      operations: ["search", "read", "grep"]
      write_access: true
  - files:
      create: true
      edit: true
      delete: true (Phase 4 only)
  - commands:
      execute: true
      sandbox: true
      whitelisted_commands:
        - "npm run test"
        - "npm run build"
        - "npm run lint"
        - "npm run format"
        - "git add/commit/push"
        - "docker build/run" (if applicable)
  - test_runner:
      run: true
      destroy_data: false
  - build_system:
      transpile: true
      compile: true
      bundle: true
  - memory:
      can_read: true
      can_update: true
      update_triggers:
        - "On pattern discovered"
        - "On error/lesson learned"
        - "On Phase completion"
```

### What Developer Can Do ✅

- Read all code and documentation
- Create new files following conventions
- Modify files according to spec
- Write and run tests
- Run build and lint commands
- Commit code changes
- Update `.memory.md` with learnings
- **DURING PHASE 4 ONLY**: Delete files if necessary
- Ask for clarification/approval before risky changes

### What Developer Cannot Do ❌

- Modify files NOT in the current spec
- Access production databases
- Modify `.memory.md` arbitrary sections
- Delete files in Phase 2 (only Phase 4)
- Access external APIs directly
- Change other developers' code without review
- Merge to protected branches
- Deploy to production

### Developer Workflow

```
@developer: Implement OAuth system (from spec)
       ↓
Load spec from Phase 1 ✓
       ↓
Phase 2a (Red-Green-Blue):
  1. Write failing tests ✓
  2. Implement code ✓
  3. Refactor ✓
  4. Run tests: PASS ✓
  5. Run linter: PASS ✓
  6. Commit ✓
       ↓
Repeat for each task in spec
       ↓
All acceptance criteria passing ✓
All commits clean ✓
       ↓
Ready for @reviewer
```

---

## 🔍 Reviewer Role: Quality Mode

### Available Tools

```yaml
role: "reviewer"
mode: "read-only"
session_duration: "30-45 minutes"
tools:
  - codebase:
      operations: ["search", "read", "grep"]
      write_access: false
  - git:
      operations: ["diff", "log", "show"]
      write_access: false
  - tests:
      run: true
      modify: false
  - problems:
      can_view: true
      can_dismiss: false
  - documentation:
      can_read: true
      can_write: false
  - memory:
      can_read: true
      can_update: false
```

### What Reviewer Can Do ✅

- Read all code changes
- View git diffs and history
- Run test suite (read-only)
- Run static analysis
- Access documentation
- Read security guidelines
- Check against acceptance criteria
- **Identify missing e2e tests** and document required scenarios
- **ONLY OUTPUT**: Findings document (including e2e gaps), approval/rejection decision

### What Reviewer Cannot Do ❌

- Modify code (even typos)
- Run commands that modify state
- Update `.memory.md`
- Approve on behalf of user
- Make exceptions to standards
- Assume changes not in spec are correct

### Reviewer Workflow

```
@reviewer: Review OAuth implementation against spec
       ↓
Load spec from Phase 1 ✓
Load code changes from Phase 2 ✓
       ↓
Check each acceptance criterion ✓
  - User can login with GitHub? YES ✓
  - Token stored encrypted? YES ✓
  - etc.
       ↓
Security review: Check for...
  - Input validation ✓
  - No hardcoded secrets ✓
  - SQL injection prevention ✓
  - etc.
       ↓
Code quality check:
  - SOLID principles ✓
  - Clean Architecture ✓
  - etc.
       ↓
If all pass: APPROVED ✓
If issues: Document findings → Phase 4
```

---

## 👤 User Role: Decision Authority

### Available Interactions

```yaml
role: "user"
authority: "decision_maker"
responsibilities:
  - approve_specifications
  - request_changes
  - approve_implementations
  - manage_scope_changes
  - handle_exceptions
```

### What User Can Do ✅

- Provide requirements and context
- Approve/reject specifications
- Request changes at any phase
- Ask questions and clarifications
- Make final decisions on trade-offs
- Handle edge cases and exceptions
- Decide on scope changes

### What User Cannot Be Automated ❌

- User decisions cannot be automated
- At each gate, only user can approve
- AI can recommend, but user decides

### User Approval Gates

```
Gate 1 (After Phase 1): Approve spec
Gate 2 (After Phase 2): Confirm implementation complete
Gate 3 (After Phase 3): Approve code or request changes
Gate 4 (After Phase 5): Approve for deployment
```

---

## 🔄 Tool Boundary Examples

### Example 1: Preventing Scope Creep

```
Developer: "Should I also fix the password reset system?"
Reviewer: "No - that's out of scope per spec"
Developer: ✓ Respects boundary, focuses only on OAuth

Tools prevent this by:
- Spec is read-only for developer
- Developer can't unilaterally expand scope
- Phase 4 (iterate) is ONLY for spec-related issues
```

### Example 2: Preventing Security Issues

```
Developer: "I'll store the OAuth token in localStorage"
Reviewer: "No - needs to be in HttpOnly cookie"
Developer: Updates code, recommits

Tools prevent this by:
- Reviewer can read code (security review)
- Reviewer can see tokens are plain text
- Reviewer approves/rejects before merge
- Developer can't bypass security without approval
```

### Example 3: Preventing Data Loss

```
Developer (Phase 2): "Let me delete the old auth table"
Developer: ✓ Respects boundary - phase 2 has no delete
Developer (Phase 4): "Now I need to delete old table"
Developer: ✓ Allowed - Phase 4 (iterate) permits cleanup

Tools prevent this by:
- Delete files tool only available in Phase 4
- Phase 4 is specifically for cleanup after review
- Accidental deletions in implementation phase prevented
```

### Example 4: Preventing Unauthorized Changes

```
Developer (another feature): "Let me also fix this OAuth bug"
Developer: ✗ Cannot - bug is out of current spec
Developer: Needs new spec or separate task

Tools prevent this by:
- Spec is source of truth
- Developer can only modify spec'd files
- Out-of-scope changes blocked
- Maintains focus and traceability
```

---

## 🚨 What Happens If Tools Are Violated?

### Violation: Developer Tries to Access Database

```
Developer attempts: "db.deleteUser(123)"
System response: ❌ Database write access denied
Developer gets: "This operation requires human approval"
Next step: Developer adds flag to spec, user approves change
```

### Violation: Reviewer Tries to Modify Code

```
Reviewer attempts: "Fix typo in OAuth.ts"
System response: ❌ File edit access denied for reviewer role
Reviewer gets: "Generate finding, developer will fix in Phase 4"
Next step: Reviewer documents finding, developer fixes
```

### Violation: Architect Tries to Execute Commands

```
Architect attempts: "npm run test"
System response: ❌ Command execution denied in architect mode
Architect gets: "Only read-only operations allowed"
Next step: Architect continues with planning
```

---

## 🛡️ Defense in Depth

### Layer 1: Role-Based Access Control
- Each role has specific tool permissions
- Tools verify role before execution

### Layer 2: Phase Boundaries
- Phase 2 (implementation) ≠ Phase 4 (iteration)
- Delete operations only allowed in Phase 4

### Layer 3: Spec Enforcement
- Developer must reference spec
- Can only modify files listed in spec
- Prevents scope creep

### Layer 4: Validation Gates
- User approval at each phase end
- Prevents accidental escalation

### Layer 5: Memory Management
- Patterns documented in `.memory.md`
- Prevents repeating security mistakes

### Layer 6: Audit Trail
- All changes tracked in git
- All decisions tracked in `.memory.md`
- Full traceability for compliance

---

## 📋 Tool Boundaries Checklist

### Before Starting Phase 1 (Architect)

- [ ] User provides feature request
- [ ] Architect has read-only access only
- [ ] Architect cannot execute code
- [ ] Architect generates markdown spec only
- [ ] User must approve spec before Phase 2

### Before Starting Phase 2 (Developer)

- [ ] Spec is approved and locked
- [ ] Developer loads spec as reference
- [ ] Developer has full file/command access (sandboxed)
- [ ] Developer follows spec exactly
- [ ] Developer commits frequently with clear messages

### Before Starting Phase 3 (Reviewer)

- [ ] Developer has completed all spec items
- [ ] Reviewer has read-only access only
- [ ] Reviewer checks against spec
- [ ] Reviewer runs tests (read-only)
- [ ] Reviewer either approves or documents findings

### Before Starting Phase 4 (Iterate)

- [ ] Reviewer has documented findings
- [ ] User approves changes needed
- [ ] Developer goes back to Phase 2 flow
- [ ] Changes must address reviewer findings
- [ ] Loop back to Phase 3 (new review session)

### Before Starting Phase 5 (Verify)

- [ ] All reviewers approve
- [ ] All changes complete
- [ ] Developer runs final verification
- [ ] All checks passing
- [ ] Ready for merge/deployment

---

## 🔧 Configuring Tool Boundaries

Tools are defined in `.github/chatmodes/` and `.github/apm.yml`:

### In Chatmode Files

```yaml
# .github/chatmodes/developer.chatmode.md
tools:
  allowed:
    - codebase: "search, read, grep"
    - files: "create, edit, delete (Phase 4 only)"
    - commands: "run (whitelisted commands only)"
    - tests: "run (read-only database)"
  forbidden:
    - network: "No external API calls"
    - secrets: "No credential access"
    - admin: "No system administration"
```

### In APM Configuration

```yaml
# .github/apm.yml
mcp:
  enabled: true
  security_level: "strict"
  allowed_protocols:
    - github: "Repository and issue access"
    - filesystem: "Safe file operations only"
    - shell: "Sandboxed command execution"
```

---

## 📚 Related Documentation

- [APM Configuration](.github/apm.yml) - Tool boundaries config
- [Development Cycle](.github/prompts/development-cycle.prompt.md) - Phase-based workflow
- [Architect Chatmode](.github/chatmodes/architect.chatmode.md) - Planning role
- [Developer Chatmode](.github/chatmodes/developer.chatmode.md) - Implementation role
- [Reviewer Chatmode](.github/chatmodes/reviewer.chatmode.md) - QA role
- [Inner/Outer Loop](.github/INNER-OUTER-LOOP.md) - Workflow modes

---

## 💡 Key Takeaway

**MCP tool boundaries are not restrictions—they're guardrails that enable confident automation.**

By defining exactly what each role can do:
✅ We prevent mistakes before they happen  
✅ We keep security intact  
✅ We maintain audit trails  
✅ We scale safely from 1 developer to 100+  
✅ We enable production-grade AI workflows  

