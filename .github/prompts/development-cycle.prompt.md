---
mode: "agent"
description: "Orchestrated development cycle: Architect → Developer → Reviewer → iterate"
tools: ["codebase", "search", "editFiles", "runCommands", "problems", "testFailure"]
autoExecute: true
---

# 🔄 Development Cycle Orchestration

This workflow orchestrates the complete development cycle through distinct phases, each with its own focus and validation gates.

## IMMEDIATE ACTION REQUIRED

Before proceeding, you MUST:
1. **DETECT YOUR OPERATING SYSTEM** (required for command compatibility)
2. Ask the user for the feature request if not provided
3. Create and save an architecture document (.spec.md)
4. Create and save a task breakdown document
5. **Recommend session splitting**: This workflow is optimized for SEPARATE sessions per phase

---

## 🖥️ OPERATING SYSTEM DETECTION

**YOU MUST DETECT YOUR OS NOW** - All commands in this workflow are OS-specific:

```bash
# Detect OS and set command variables
# Run ONE of these commands based on your system:

# Windows (PowerShell)
$OS = "Windows"
$SHELL_CMD = "powershell"
$LIST_FILES = "Get-ChildItem"
$MKDIR = "New-Item -ItemType Directory"
$DELETE = "Remove-Item -Recurse -Force"

# macOS / Linux (Bash)
OS="Unix"
SHELL_CMD="bash"
LIST_FILES="ls -la"
MKDIR="mkdir -p"
DELETE="rm -rf"
```

**Your detected OS:**
- [ ] **Windows** (PowerShell) - Use PowerShell commands
- [ ] **macOS** (Bash) - Use Unix/bash commands
- [ ] **Linux** (Bash) - Use Unix/bash commands

### Command Reference by OS

Throughout this workflow, commands appear in both formats:

```
Windows (PowerShell):    [PowerShell syntax]
Unix/Linux/macOS (Bash):  [bash syntax]
```

**Example format you'll see:**
```
# Windows: Get-ChildItem
# Unix:    ls -la
```

Select the command variant that matches YOUR operating system.

**CRITICAL**: Copy/paste the exact command for your OS. Do NOT mix command styles.

---

---

## 🎯 Context Engineering: Session Splitting

This workflow follows GitHub's **agentic primitives framework** with session splitting for fresh context:

| Phase | Chat Mode | Session | Focus |
|-------|-----------|---------|-------|
| Phase 1 | `@architect` | New session | Design only - no implementation |
| Phase 2 | `@developer` | New session | Code & tests - no planning |
| Phase 3 | `@reviewer` | New session | Review only - no coding |
| Phase 4 | `@developer` | New session | Fixes only - iterate as needed |
| Phase 5 | `@developer` | Same session | Verify & complete |

**Why session splitting?** Fresh context = better focus = fewer mistakes. Each phase gets a clean context window.

### Memory-Driven Development
Your agent has access to:
- **[.memory.md](.memory.md)** - Patterns, decisions, and lessons learned
- **[Architecture context](../instructions/)** - Domain patterns specific to this project
- **[Chatmode expertise](.chatmodes/)** - Role-specific knowledge boundaries

## Input Required

```yaml
Feature: [Brief description of what to build]
Context: [Any relevant background or constraints]
Priority: [High/Medium/Low]
```

---

# Phase 1: 🏗️ ARCHITECT

> **Role**: System design and planning specialist
> **Focus**: Design, don't implement

## 1.1 Requirements Analysis

1. Parse and clarify the feature request
2. Identify functional and non-functional requirements
3. List assumptions and open questions

## 1.2 Context Gathering

Review existing architecture:
- [Project architecture](../../docs/architecture.md)
- [Coding conventions](../../docs/conventions.md)
- Search for similar patterns in codebase

## 1.3 Domain Modeling (DDD)

Define the domain model:
- **Entities**: Objects with identity and lifecycle
- **Value Objects**: Immutable objects defined by attributes
- **Aggregates**: Cluster of entities with a root
- **Domain Events**: Important occurrences in the domain
- **Bounded Context**: Where does this feature live?

## 1.4 Technical Design

Create the implementation blueprint:

```markdown
### Components to Create/Modify
1. [Component] - [Purpose]
2. [Component] - [Purpose]

### API Design (if applicable)
- Endpoint: [METHOD /path]
- Request: [Schema]
- Response: [Schema]

### Data Model Changes
- [Table/Collection changes]

### Dependencies
- Internal: [modules]
- External: [packages]
```

## 1.5 Task Breakdown

Split into implementable tasks with estimates:

```markdown
### Tasks
- [ ] Task 1: [Description] (~X hrs)
- [ ] Task 2: [Description] (~X hrs)
- [ ] Task 3: [Description] (~X hrs)

### Suggested Order
1. Domain entities and value objects
2. Use cases / application services  
3. Infrastructure (repositories, external services)
4. API layer (controllers, validators)
5. Integration tests
```

## 1.6 Memory Integration

Before finalizing design, check:
- [Previous patterns](.memory.md#successful-implementations) - Reuse what worked
- [Failed approaches](.memory.md#failed-approaches-dont-repeat) - Avoid past mistakes
- [Architectural decisions](.memory.md#key-decisions-made) - Maintain consistency

---

## 🚨 GATE 1: Architecture Review

### YOU MUST CREATE THESE FILES:

1. **`docs/[feature-name].spec.md`** - Complete specification using the [spec template](../specs/_template.spec.md)
   - Problem statement
   - Functional & non-functional requirements
   - Technical design with architecture diagram
   - Task breakdown with estimates
   - Acceptance criteria

2. **Update `.memory.md`** - Add to [Key Decisions Made](.memory.md#key-decisions-made):
   - Date, decision made, and rationale

### THEN PRESENT TO USER:
1. Domain model diagram/description (from spec)
2. Technical design overview (from spec)
3. Task breakdown with estimates (from spec)
4. Identified risks and mitigations (from spec)
5. Open questions needing clarification

**WAIT FOR USER APPROVAL before proceeding to Phase 2 (DEVELOPER mode).**

DO NOT PROCEED UNTIL USER EXPLICITLY APPROVES THE SPECIFICATION.

---

# Phase 2: 👨‍💻 DEVELOPER

> **Role**: Implementation specialist (TDD, Clean Architecture, SOLID)
> **Focus**: Build what was designed, test-first
> **Chatmode**: `@developer` - New session with fresh context
> **Input**: Approved specification from Phase 1 (`docs/[feature-name].spec.md`)

## 2.1 Setup

**Start fresh session:**
1. Use `@developer` chat mode (different session from architect)
2. Input: "I have an approved specification at `docs/[feature-name].spec.md`. Implement it following TDD."
3. Reference the spec file, not just memory

For each task from the specification:
1. Understand the acceptance criteria
2. Identify the test cases needed
3. Locate where code should live (Clean Architecture layers)

## 2.2 YOU MUST WRITE TEST FILES FIRST

**CRITICAL: Tests must be created and written to disk BEFORE any implementation**

**BUT ALSO CRITICAL: Tests must actually test the behavior, not just existence**

Bad test (doesn't catch bugs):
```typescript
it('should create user', () => {
  const user = new User('test@example.com');
  expect(user).toBeDefined();  // ❌ Passes even if constructor broken!
});
```

Good test (catches real issues):
```typescript
it('should create user with valid email', () => {
  const user = new User('test@example.com');
  expect(user.email).toBe('test@example.com');  // ✅ Checks actual value
  expect(user.isValidated()).toBe(false);       // ✅ Checks behavior
});

it('should throw error with invalid email', () => {
  expect(() => new User('notanemail')).toThrow(InvalidEmailError);  // ✅ Tests error case
});

it('should reject empty email', () => {
  expect(() => new User('')).toThrow();  // ✅ Tests edge case
});
```

**Testing Principle**: Your test should FAIL if the software doesn't work, even if the code looks correct.

For each test:
- Ask: "If the implementation was completely wrong, would this test catch it?"
- If answer is NO → Rewrite the test

**Example structure**:
```typescript
describe('UserEntity', () => {
  describe('constructor', () => {
    it('should create user with valid email', () => {
      // Arrange
      const email = 'test@example.com';
      
      // Act
      const user = new UserEntity(email);
      
      // Assert
      expect(user.email).toBe(email);
    });

    it('should throw error with invalid email', () => {
      expect(() => new UserEntity('invalid')).toThrow();
    });
  });
});
```

## 2.3 TDD Implementation Loop

For each component:

```
┌─────────────────────────────────────────┐
│  1. WRITE TEST FILE (Red)               │
│     ✅ CREATE and SAVE test file        │
│     - Run test, confirm it FAILS        │
├─────────────────────────────────────────┤
│  2. IMPLEMENT CODE FILE (Green)         │
│     ✅ CREATE and SAVE implementation   │
│     - Write just enough to pass         │
│     - No extra features                 │
│     - Run test, confirm it PASSES       │
├─────────────────────────────────────────┤
│  3. REFACTOR (Blue)                     │
│     - Clean up code                     │
│     - Apply SOLID principles            │
│     - Run tests, confirm still GREEN    │
└─────────────────────────────────────────┘
         ↓ Repeat for next test case
```

## 2.4 Implementation Checklist

**BEFORE PROCEEDING**: Create and save ALL test files for the component
- [ ] Test file created and saved to disk
- [ ] Test file imports/references implementation (even if not created yet)
- [ ] Tests implement acceptance criteria from spec
- [ ] Run tests, they FAIL as expected (Red phase)

**THEN** implement the code:
- [ ] Implementation file created and saved
- [ ] All tests passing (Green phase)
- [ ] Domain logic in domain layer (no framework dependencies)
- [ ] Use cases orchestrate domain operations
- [ ] Infrastructure implements interfaces (dependency inversion)
- [ ] No business logic in controllers
- [ ] Error handling implemented
- [ ] Input validation at boundaries
- [ ] All acceptance criteria from spec verified

## 2.5 Memory Update

During development:
- Document any patterns you discover in [.memory.md](.memory.md#successful-implementations)
- Log any issues encountered in [Known Issues](.memory.md#known-issues--quirks)

## 2.6 Documentation

- Add JSDoc/docstrings to public APIs
- Add comments explaining test cases in test files
- Update API documentation if endpoints changed
- Note any deviations from the specification

---

## 🚨 GATE 2: Implementation Complete

### YOU MUST HAVE CREATED:

**Test Files** (MUST exist first):
- [ ] `src/[layer]/__tests__/[component].test.ts` for each component
- [ ] All test files written to disk
- [ ] All tests passing (run test suite to verify)

**Implementation Files** (created after tests):
- [ ] `src/[layer]/[component].ts` for each component  
- [ ] All files written to disk
- [ ] All tests still passing

**Documentation Files**:
- [ ] Updated JSDoc in all files
- [ ] Updated API documentation if needed

### VERIFICATION BEFORE PRESENTING:
1. List all test files created
2. List all implementation files created
3. Run full test suite and report: ✅ PASSING / ❌ FAILING
4. Show test coverage percentage

### THEN PRESENT TO USER:
1. All files created (test + implementation with paths)
2. Test suite status (PASSING / FAILING - with details if failing)
3. Test coverage percentage
4. Any deviations from architecture plan
5. Questions or concerns for reviewer

**WAIT FOR USER APPROVAL before proceeding to Phase 3 review.**

DO NOT PROCEED if tests are failing or test files weren't created.

---

# Phase 3: 🔍 REVIEWER

> **Role**: Quality assurance specialist
> **Focus**: Find issues, don't fix them (yet)
> **Chatmode**: `@reviewer` - New session with fresh context
> **Input**: Implemented code from Phase 2

## 3.1 Start Fresh Session

Use `@reviewer` chat mode to trigger:
- Different context window from developer
- Review-focused tool access
- Security and architecture compliance checks

Check each file for:

### Correctness
- [ ] Logic handles all cases correctly
- [ ] Edge cases considered
- [ ] Error handling appropriate

### Clean Architecture Compliance
- [ ] Dependencies point inward only
- [ ] Domain layer has no framework imports
- [ ] Use cases don't access infrastructure directly
- [ ] Controllers are thin (no business logic)

### SOLID Principles
- [ ] **S**: Each class has single responsibility
- [ ] **O**: Extensions don't require modifications
- [ ] **L**: Subtypes are substitutable
- [ ] **I**: Interfaces are focused and minimal
- [ ] **D**: High-level modules don't depend on low-level

### TDD Quality
- [ ] Tests exist for all public behavior
- [ ] Tests are meaningful (not just coverage padding)
- [ ] Tests follow AAA pattern (Arrange, Act, Assert)
- [ ] Edge cases and errors are tested

### E2E Test Coverage
- [ ] Critical user paths have e2e tests ✓
- [ ] E2E tests verify feature works end-to-end (not just unit logic)
- [ ] E2E tests cover happy path AND error scenarios
- [ ] If e2e tests missing: Document required test scenarios
- [ ] Example e2e scenarios to verify:
  - Complete user workflow from start to finish
  - API integration works with database
  - UI reflects data changes correctly
  - Error handling works (invalid input, network failures)

### Security
- [ ] Input validation present
- [ ] No sensitive data exposure
- [ ] Authentication/authorization enforced
- [ ] No injection vulnerabilities

### Performance
- [ ] No obvious N+1 queries
- [ ] Appropriate use of caching
- [ ] No unnecessary computations

## 3.2 Review Findings Format

```markdown
### 🔴 Critical (Must Fix)
- [File:Line] Issue description → Suggested fix

### 🟠 Major (Should Fix)
- [File:Line] Issue description → Suggested fix

### 🟡 Minor (Consider)
- [File:Line] Issue description → Suggested fix

### 🟢 Positive Feedback
- What was done well

### E2E Test Gaps (if found)
If e2e tests are missing or incomplete, document:
- Which critical user paths lack e2e coverage
- What the e2e test should verify
- Example: "Missing e2e test for user signup flow: email validation → confirmation → redirect"
- Severity: [Critical / Major / Minor]
```

---

## 🚨 GATE 3: Review Complete

### YOU MUST NOW:

1. **Create review document**: `docs/[feature-name].review.md`
   - Include findings, recommendations, approval status

2. **CRITICAL: Update `.memory.md`** - Prevent repeated issues:

   **For EACH issue found:**
   - Add to `.memory.md#known-issues--quirks` section
   - Include: Category, Description, Root Cause, How to Detect, How to Prevent
   - Example:
     ```
     ### Issue: Missing validation on email field
     - **Category**: Testing/Security
     - **Description**: Email field accepts invalid formats
     - **How to Detect**: Add test: invalid email should throw
     - **How to Prevent**: Validate all string inputs at boundaries
     - **Status**: Open - Will fix in Phase 4
     ```

   **For patterns:**
   - Good patterns → Add to `.memory.md#important-patterns` (reuse)
   - Bad patterns → Add to `.memory.md#failed-approaches-dont-repeat`

   **Link tests to issues:**
   - If issue X was found, add: "Test suggestion: [specific test to catch this]"
   - Phase 4 will use these test suggestions

3. **Write findings to disk**

4. **Determine outcome**: APPROVE or REQUEST CHANGES

### PRESENT TO USER:
1. Review findings by severity (🔴🟠🟡🟢)
2. SOLID/Clean Architecture compliance score
3. Test coverage assessment:
   - Unit tests: [GOOD / NEEDS WORK]
   - Integration tests: [GOOD / NEEDS WORK]
   - **E2E tests: [GOOD / MISSING / INCOMPLETE]** ⭐ NEW
4. Security assessment
5. **E2E Test Gaps** (if applicable):
   - Missing scenarios: [List required e2e tests]
   - Critical paths not covered: [Which user workflows]
   - How to fix: Pass to Phase 4 with specific e2e test requirements
6. **RECOMMENDATION: APPROVE / REQUEST CHANGES**

**If APPROVE**: Proceed to Phase 5 (Verification)
**If REQUEST CHANGES**: Proceed to Phase 4 (Iterate)

---

# Phase 4: 🔧 ITERATE (if needed)

> **Chatmode**: `@developer` - New session
> **Input**: Issues from `.memory.md#known-issues--quirks` + review findings
> **Goal**: Test-first fixes that prevent recurrence

If reviewer found issues:

## 4.1 Prevent Future Recurrence - Check Memory First

**BEFORE fixing, check `.memory.md#known-issues--quirks` for each issue:**
- Has this issue (or similar) appeared before?
- What was the prevention strategy last time?
- Are there recommended tests to catch it?
- Copy test suggestion from memory if available

## 4.2 Address Critical Issues First (Test-First)

For each critical/major issue:

1. **Reference memory**: Check if similar issue type appeared before
   - Reuse prevention strategy from `.memory.md`
   - Use test suggestion from `.memory.md`
2. **Write test first**: Use memory recommendation or create new
   - Test MUST expose the current issue
   - Confirm test FAILS with current code
3. **Fix the issue**: Apply prevention strategy
   - Confirm test now PASSES
4. **Run full test suite**: No regressions
5. **Update `.memory.md`**: Document what fixed it
   - How was it fixed?
   - Will this prevent similar issues?

## 4.3 Return to Phase 3 (Review Again)

After fixing all critical/major issues:
1. Update `.memory.md` with fix results
2. Request re-review of changed areas
3. Return to **Phase 3 (@reviewer)** for re-review in new session

**Key Principle**: The same issue type should NOT appear twice

---

# Phase 5: ✅ VERIFY & COMPLETE

> **CRITICAL**: This phase is NOT optional. You MUST actually run the software and verify it works.
> Tests passing ≠ Software working. Broken software with passing tests = test failure.

When reviewer approves, **verify the application actually works end-to-end before completing**.

## 5.1 Pre-Runtime Checks

### Step 1: Static Analysis
```
Windows (PowerShell):
  npm run lint
  npm run type-check

Unix/Linux/macOS (Bash):
  npm run lint
  npm run type-check

Python (both OS):
  pylint src/
  mypy src/
```

- [ ] No lint errors
- [ ] No type errors
- [ ] No compiler warnings (treat as errors)

### Step 2: Test Suite
```
Windows (PowerShell):
  npm test
  npm run test:coverage

Unix/Linux/macOS (Bash):
  npm test
  npm run test:coverage

Python (both OS):
  pytest
  pytest --cov=src/
```

- [ ] Unit tests: PASS ✅
- [ ] Integration tests: PASS ✅
- [ ] E2E tests (if applicable): PASS ✅
- [ ] Code coverage: Meets requirements ✅

### Step 3: Build Verification
```
Windows (PowerShell):
  npm run build

Unix/Linux/macOS (Bash):
  npm run build

Go (both OS):
  go build

Python (both OS):
  # No build step needed, skip
```

- [ ] Build succeeds ✅
- [ ] No build warnings
- [ ] Build artifacts exist ✅

## 5.2 RUNTIME EXECUTION (MANDATORY) 🚀

### Step 4: Start & Run the Application

**This is NOT optional. You MUST start the actual software:**

```
Windows (PowerShell):
  npm start              # Node/React
  python app.py          # Python/Flask (in PowerShell)
  go run main.go         # Go
  dotnet run             # .NET
  python -m flask run    # Flask

Unix/Linux/macOS (Bash):
  npm start              # Node/React
  python app.py          # Python/Flask
  go run main.go         # Go
  dotnet run             # .NET
  python -m flask run    # Flask
  ruby app.rb            # Ruby
```

**Select the command for your OS AND tech stack from above.**

**Verify:**
- [ ] Application starts successfully (no startup errors)
- [ ] Application runs without crashing
- [ ] Console/logs show no runtime errors ❌ MUST BE CLEAN
- [ ] Application stays running (doesn't crash after 5 seconds)
- [ ] Memory usage is reasonable (no memory leaks)

### Step 5: Feature Verification (User Perspective)

**YOU MUST ACTUALLY TEST THE FEATURE - Not just read the code:**

```
Windows (PowerShell):
  # For Backend API:
  curl http://localhost:3000/api/users
  # Or use PowerShell:
  Invoke-WebRequest http://localhost:3000/api/users

  # For Frontend:
  # Open http://localhost:3000 in browser
  # Click the new feature button

  # For CLI:
  .\cli-tool.exe new-command --flag
  # Or if Python:
  python cli.py new-command --flag

Unix/Linux/macOS (Bash):
  # For Backend API:
  curl http://localhost:3000/api/users

  # For Frontend:
  # Open http://localhost:3000 in browser
  # Click the new feature button

  # For CLI:
  ./cli-tool new-command --flag
```

**Select commands appropriate for your OS and application type.**

**Verify: ✅ Returns 200 OK with data**
**❌ FAIL if: 500 error, timeout, empty response, wrong format**

**Test These Scenarios:**
- [ ] **Happy Path**: Normal user action works ✅
  - Example: Create user → User created → Can retrieve it
- [ ] **Error Cases**: Invalid input handled gracefully ✅
  - Example: Invalid email → Error message, not crash
- [ ] **Edge Cases**: Boundary conditions work ✅
  - Example: Empty list → Shows "No items", not null pointer exception
- [ ] **Performance**: Feature responds within acceptable time ✅
  - Example: API returns in < 500ms
- [ ] **Data Integrity**: Data is correct after operations ✅
  - Example: Create then read → Same data returned
- [ ] **No Side Effects**: Feature doesn't break other features ✅
  - Example: Old features still work after new feature deployed

### Step 6: Browser/Client Verification (if UI)

```javascript
// Open DevTools console (F12)
// Verify: ✅ COMPLETELY CLEAN - no errors, no warnings
// ❌ FAIL if any of these appear:
//    - Error: [any error message]
//    - Uncaught TypeError
//    - 404 for resources
//    - CORS errors
//    - Deprecation warnings
```

- [ ] No console errors ✅
- [ ] No console warnings ✅
- [ ] Network requests successful (no 4xx/5xx status codes) ✅
- [ ] UI renders correctly ✅
- [ ] Interactions responsive (no lag) ✅

### Step 7: Data Verification

**Actually check the data:**

```
Windows (PowerShell):
  # Database (SQL Server / MySQL)
  sqlcmd -S localhost -U sa -P password -Q "SELECT * FROM users WHERE id = 123;"
  # Or MySQL:
  mysql -h localhost -u user -p -e "SELECT * FROM users WHERE id = 123;"

  # File system
  Get-ChildItem -Path C:\path\to\data\ -Recurse

  # API response
  Invoke-WebRequest http://localhost/api/data | ConvertFrom-Json

Unix/Linux/macOS (Bash):
  # Database (PostgreSQL / MySQL)
  psql -h localhost -U user -d database -c "SELECT * FROM users WHERE id = 123;"
  # Or MySQL:
  mysql -h localhost -u user -p -e "SELECT * FROM users WHERE id = 123;"

  # File system
  ls -la /path/to/data/

  # API response
  curl http://localhost/api/data | jq
```

**Select the database command for your system (PostgreSQL, MySQL, SQL Server, MongoDB, etc.)**

- [ ] Data persisted correctly ✅
- [ ] No missing fields ✅
- [ ] No corrupted data ✅
- [ ] Relationships intact (if applicable) ✅

### Step 8: Load Test (if applicable)

```
Windows (PowerShell):
  # Run concurrent requests (simple load test)
  # Does software handle multiple users?
  $jobs = @()
  for ($i = 1; $i -le 100; $i++) {
    $jobs += Start-Job -ScriptBlock {
      Invoke-WebRequest http://localhost/api/feature
    }
  }
  $jobs | Wait-Job
  $jobs | Receive-Job

Unix/Linux/macOS (Bash):
  # Run concurrent requests (simple load test)
  # Does software handle multiple users?
  for i in {1..100}; do
    curl http://localhost/api/feature &
  done
  wait
```

**Select the load test script appropriate for your OS.**

- [ ] Can handle multiple concurrent requests ✅
- [ ] No race conditions ✅
- [ ] No resource exhaustion ✅

---

## 🚨 GATE 4: Verification Complete - SOFTWARE ACTUALLY WORKS

### YOU MUST NOW:

1. **ACTUALLY RUN THE SOFTWARE** - Not just read the code
   - [ ] Started application successfully
   - [ ] Tested feature in running software
   - [ ] Verified in browser/CLI/API
   - [ ] Checked console/logs (CLEAN)
   - [ ] Tested error cases (not crash)

2. **Create verification report**: `docs/[feature-name].verification.md`
   ```markdown
   # Verification Report: [Feature Name]
   
   ## Runtime Execution
   - [x] Application started: npm start → SUCCESS
   - [x] Application running: No crashes for 5+ minutes
   - [x] Console clean: No errors, no warnings
   
   ## Feature Testing
   - [x] Happy path: User creates account → Account created ✅
   - [x] Error handling: Invalid email → Shows error message ✅
   - [x] Edge cases: Empty list → Shows "No items" ✅
   
   ## Data Verification
   - [x] Data persisted to database ✅
   - [x] Retrieved data matches input ✅
   - [x] No data corruption ✅
   
   ## Performance
   - [x] API response time: 150ms (acceptable)
   - [x] UI renders: <500ms (acceptable)
   - [x] No lag on interactions ✅
   
   ## Browser Console (if UI)
   - [x] No errors
   - [x] No warnings
   - [x] Network requests successful
   
   ## Concurrent Load Test
   - [x] Handled 100 concurrent requests
   - [x] No race conditions
   - [x] No resource exhaustion
   
   ## Final Status
   ✅ VERIFIED - Software works as designed
   ```

3. **Include actual evidence:**
   - Screenshot of working feature
   - Console output (clean)
   - API response samples
   - Database query results
   - Performance metrics

4. **Determine outcome**: 
   - ✅ VERIFIED: Software works, proceed to completion
   - ❌ NOT VERIFIED: Return to Phase 4 or Phase 2 (code has issues)

### FAIL CRITERIA (Return to Development):
- [ ] Application crashes on startup
- [ ] Feature doesn't work when tested
- [ ] Console errors or warnings
- [ ] Unhandled exceptions during testing
- [ ] Data corruption or loss
- [ ] Performance unacceptable
- [ ] Error cases cause crashes

### IF VERIFICATION FAILS:

**This means the tests weren't good enough.** Return to Phase 2:
1. Identify: Why didn't tests catch this?
2. Add test cases that would catch this failure
3. Re-implement with better tests
4. Run full cycle again

**Update `.memory.md` with:**
- Issue: "Tests passed but software didn't work - [specific issue]"
- Root cause: "Test didn't cover [scenario]"
- Prevention: "Always test in running software, not just tests"

**Key Point**: A test suite that allows broken software to pass = test suite failure

### IF ANY STEP FAILS:
- Classify the issue (Architecture / Implementation / Configuration)
- Route to appropriate phase based on issue type
- Provide specific instructions for fixing
- Do NOT skip verification - run again after any fix

### IF ALL STEPS PASS:
- Proceed to 5.2 Final Checklist
- Update `.memory.md` with lessons learned
- Commit changes with appropriate message

---

## 5.2 Final Checklist
- [ ] All tests passing (verified in 5.1)
- [ ] No lint/type errors (verified in 5.1)
- [ ] Application builds successfully (verified in 5.1)
- [ ] Application runs without errors (verified in 5.1)
- [ ] Documentation updated
- [ ] Architecture decisions documented in `.memory.md`
- [ ] Ready for commit

## 5.3 Execute Completion Actions

YOU MUST NOW:
1. **Create summary document**: `docs/[feature-name].completion.md`
   - Include all completion checklist items
   - Reference all created files
   - Document key decisions and patterns
2. **Update `.memory.md`** with:
   - Patterns used that worked well
   - Decisions made and rationale
   - Lessons learned
3. **Write all files to disk**
4. **Run git commands** to commit changes

## 5.4 Commit Strategy
```bash
# If single logical change:
git commit -m "feat(domain): implement [feature name]"

# If multiple logical changes, commit separately:
git commit -m "feat(domain): add [Entity] aggregate"
git commit -m "feat(app): add [UseCase] use case"
git commit -m "feat(api): add [endpoint] endpoint"
```

## 5.5 Final Summary

Present to user:
- ✅ Feature successfully implemented
- 📁 All files created and committed
- 📊 Verification report available
- 📚 Documentation updated
- 💾 Changes saved to repository

---

# 📊 Cycle Summary Template

```markdown
## Feature: [Name]

### Phase 1: Architecture ✓
- Designed: [brief summary]
- Tasks: [X] identified

### Phase 2: Development ✓
- Implemented: [X] components
- Tests: [X] passing

### Phase 3: Review
- Issues found: [X critical, Y major, Z minor]
- Status: [APPROVED / ITERATING]

### Phase 4: Iteration
- Fixes applied: [X]
- Re-review: [PASSED / PENDING]

### Phase 5: Verification
- Static analysis: [PASS / FAIL]
- Test suite: [PASS / FAIL]
- Build: [PASS / FAIL]
- Runtime: [PASS / FAIL]
- Smoke test: [PASS / FAIL]
- Status: [VERIFIED / FIXING]

### Completion
- Committed: [hash]
- Documented: [Yes/No]
- Application running: ✅
```
