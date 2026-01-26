---
description: "Complete guide to agentic primitives used in this project"
purpose: "Reference for understanding and using the agentic framework"
---

# Agentic Primitives Guide

This project uses **three layers of agentic primitives** based on GitHub's framework for reliable AI workflows.

## Layer 1: Markdown Prompt Engineering

### Foundation Principles
- **Semantic structure**: Use headers, lists, and links to guide reasoning
- **Precision**: Eliminate ambiguity with specific instructions
- **Context loading**: Link to relevant files and documentation
- **Tool integration**: Specify which MCP tools to use
- **Validation gates**: Include mandatory approval checkpoints

### Primary Prompt File
- **File**: [.github/prompts/development-cycle.prompt.md](.github/prompts/development-cycle.prompt.md)
- **Purpose**: Orchestrates complete 5-phase development workflow
- **Usage**: `@workspace /prompt development-cycle` in VS Code
- **Output**: Managed specifications, tests, implementation, review

---

## Layer 2: Agent Primitives

### Agent Primitive Files

#### 1. **Instructions** (`.instructions.md`)
Apply targeted guidance based on file type or domain.

**Global Rules**:
- File: [.github/copilot-instructions.md](.github/copilot-instructions.md)
- Applies to: All files unless overridden

**Domain-Specific Rules**:
```
.github/instructions/
├── frontend.instructions.md    # applyTo: "**/*.{js,jsx,ts,tsx}"
├── backend.instructions.md     # applyTo: "**/*.{py,go,java,cs}"
├── testing.instructions.md     # applyTo: "**/test/**,**/*.test.*"
└── docs.instructions.md        # applyTo: "**/*.md"
```

**Usage**: Copilot automatically loads matching instructions based on current file.

#### 2. **Chat Modes** (`.chatmode.md`)
Create role-based agents with specific expertise and tool boundaries.

**Available Modes**:
```
.github/chatmodes/
├── architect.chatmode.md       # @architect - Design specialist
├── backend-dev.chatmode.md     # @backend - API/backend specialist
├── frontend-dev.chatmode.md    # @frontend - UI specialist
└── reviewer.chatmode.md        # @reviewer - Quality assurance
```

**Usage**: Select mode with `@mode-name` in Copilot chat, or use in workflow phases:
- Phase 1 (Architecture) → `@architect` (NEW SESSION)
- Phase 2 (Implementation) → `@developer` (NEW SESSION)
- Phase 3 (Review) → `@reviewer` (NEW SESSION)

**Why separate modes?**
- Enforces professional boundaries (architect doesn't code)
- Limits tool access (reviewer can't modify code)
- Provides specialized knowledge (backend != frontend)
- Prevents cross-domain interference

#### 3. **Workflows** (`.prompt.md`)
Orchestrate complex processes using primitives together.

**Current Workflows**:
- File: [.github/prompts/development-cycle.prompt.md](.github/prompts/development-cycle.prompt.md)
- Type: Complete development orchestration
- Phases: 5 (Architecture → Development → Review → Iterate → Verify)
- Gates: 4 validation checkpoints requiring user approval

**Features**:
- Automatic spec generation
- TDD enforcement (tests before code)
- Memory integration (patterns preserved)
- Session splitting (fresh context per phase)
- Deterministic output (files written to disk)

#### 4. **Specifications** (`.spec.md`)
Implementation-ready blueprints created by architects, consumed by developers.

**Location**: `docs/specs/`
- Template: [docs/specs/_template.spec.md](docs/specs/_template.spec.md)
- Usage**: Architect creates spec in Phase 1, developer implements from spec in Phase 2

**Sections**:
- Problem statement & proposed solution
- Functional & non-functional requirements
- Technical design & architecture
- Task breakdown with estimates
- Testing strategy
- Risk assessment
- Acceptance criteria

#### 5. **Memory** (`.memory.md`)
Preserve knowledge and patterns across sessions.

**Location**: [.memory.md](.memory.md)
**Purpose**: Compound intelligence - project learns and improves with each iteration

**Sections**:
- Key architectural decisions
- Important patterns (what works)
- Failed approaches (what doesn't)
- Known issues & quirks
- Successful implementations
- Community patterns (reusable solutions)

**Updated by**: Architect (decisions), Developer (patterns, issues), Reviewer (quality metrics)

#### 6. **Context Helpers** (`.context.md`)
Optimize information retrieval and agent focus.

**Available Contexts**:
```
.github/context/
├── architecture.context.md     # Layer structure, dependencies, file locations
└── testing.context.md          # Test patterns, coverage requirements, examples
```

**Purpose**: Provide quick reference without cluttering working context
**Usage**: Agents reference automatically when relevant to task

---

## Layer 3: Context Engineering

### Session Splitting Strategy

**Why?** LLMs have finite context windows. Fresh sessions = better focus.

```
Session 1: Architect        → Creates spec
        ↓
Session 2: Developer        → Implements from spec
        ↓
Session 3: Reviewer         → Reviews code
        ↓
Session 4: Developer        → Fixes issues (if needed)
        ↓
Session 5: Developer        → Verifies & completes
```

**In Practice**:
```powershell
# Phase 1 - Architecture (new session)
@architect
"Design a user authentication system. Follow development-cycle.prompt"

# Phase 2 - Implementation (new session, after approval)
@developer
"Implement the spec from docs/user-auth.spec.md using TDD"

# Phase 3 - Review (new session)
@reviewer
"Review the implementation in src/auth/ for quality"
```

### Memory-Driven Development

**How agents learn:**

1. **Documentation**: Patterns captured in [.memory.md](.memory.md)
2. **References**: Every phase checks memory for precedent
3. **Updates**: Each phase improves memory with new learnings
4. **Reuse**: Next feature benefits from accumulated knowledge

**Example Flow**:
- **Iteration 1**: Architect makes design decision → Documents in memory
- **Iteration 2**: Architect for new feature → Reads memory → Follows same pattern
- **Iteration 3**: Developer discovers edge case → Documents in memory
- **Iteration 4**: Developer for similar feature → Writes test for same edge case

### Modular Instructions

**Context Preservation**:
- Global rules in [copilot-instructions.md](.github/copilot-instructions.md)
- Domain-specific in [instructions/](../.github/instructions/)
- Uses `applyTo` patterns to load selectively

**Benefit**: Only relevant instructions load → More context for actual work

### Tool Boundaries

**Each chatmode gets specific tools**:
```
Architect: codebase, search, problems
Developer: codebase, search, editFiles, runCommands, problems, testFailure
Reviewer: codebase, search, problems
```

**Why?** Prevents cross-domain mistakes (architect can't code, reviewer can't modify)

---

## Complete Workflow Example

### Scenario: Build User Authentication

#### Step 1: Session 1 - Architecture (use @architect mode)
```
Command: @architect
Prompt: "Design a secure user authentication system"

Outputs created:
- docs/user-auth.spec.md (complete specification)
- .memory.md updated with architectural decisions
- User approval gate

Flow: 
  1.1 Requirements Analysis
  1.2 Context Gathering (reads .memory.md patterns)
  1.3 Domain Modeling (DDD)
  1.4 Technical Design
  1.5 Task Breakdown
  → GATE 1: User approves spec
```

#### Step 2: Session 2 - Implementation (use @developer mode)
```
Command: @developer
Prompt: "Implement the spec from docs/user-auth.spec.md"

Outputs created:
- src/domain/auth/entities/User.ts
- src/domain/auth/__tests__/User.test.ts
- src/application/auth/use-cases/...
- src/infrastructure/auth/repositories/...
- All tests passing

Flow:
  2.1 Read spec for acceptance criteria
  2.2 For each component:
    - Write failing test (Red)
    - Implement code (Green)
    - Refactor (Blue)
  2.3 Update .memory.md with patterns discovered
  → GATE 2: User approves implementation
```

#### Step 3: Session 3 - Review (use @reviewer mode)
```
Command: @reviewer
Prompt: "Review the auth implementation in src/ against SOLID"

Outputs created:
- docs/user-auth.review.md (review findings)
- .memory.md updated with patterns and issues
- Recommendation: APPROVE or REQUEST CHANGES

Flow:
  3.1 Check code quality
  3.2 Verify SOLID principles
  3.3 Check test coverage
  3.4 Security assessment
  → GATE 3: Review complete with recommendation
```

#### Step 4: Session 4 - Verification (use @developer mode)
```
Command: @developer
Prompt: "Verify the auth feature - run tests, linting, build"

Outputs created:
- Verification report
- .memory.md updated with lessons
- Feature committed to git

Flow:
  5.1 Run static analysis (lint, types)
  5.2 Run full test suite
  5.3 Build verification
  5.4 Runtime verification
  5.5 Manual smoke test
```

---

## Getting Started Checklist

✅ **Already completed in this project**:
- [x] Global instructions in `.github/copilot-instructions.md`
- [x] Domain-specific instructions in `.github/instructions/`
- [x] Chat modes configured in `.github/chatmodes/`
- [x] Main workflow in `.github/prompts/development-cycle.prompt.md`
- [x] Spec template in `docs/specs/_template.spec.md`
- [x] Memory file in `.memory.md`
- [x] Context helpers in `.github/context/`

**To use it**:
1. Open VS Code with this workspace
2. Run: `@workspace /prompt development-cycle`
3. Follow the interactive workflow
4. Use `@architect`, `@developer`, `@reviewer` modes for phase separation
5. Check `.memory.md` to see how knowledge accumulates

---

## Best Practices

### For Architects
- Always check `.memory.md` for previous decisions
- Create complete `.spec.md` with acceptance criteria
- Update memory with key decisions
- Use `@architect` mode with limited tools (no code execution)

### For Developers
- Start in new session with `@developer` mode
- Reference the `.spec.md` file, not just memory
- Implement tests BEFORE code (TDD)
- Update memory with patterns discovered
- Follow `.github/context/architecture.context.md` for structure

### For Reviewers
- Use separate session with `@reviewer` mode
- Check against SOLID, Clean Architecture, security
- Document findings in review file
- Update memory with patterns observed
- Use `@reviewer` mode with read-only tools

### For Teams
- Commit `.prompt.md`, `.spec.md`, `.memory.md` files to git
- Share patterns via `.memory.md`
- Customize `.instructions.md` for team standards
- Use chat modes to enforce professional boundaries
- Review completed `.spec.md` and `.review.md` as examples

---

**Last Updated**: January 22, 2026  
**Framework Version**: GitHub Agentic Primitives v1  
**Reference**: [GitHub Blog - Agentic Primitives](https://github.blog/ai-and-ml/github-copilot/how-to-build-reliable-ai-workflows-with-agentic-primitives-and-context-engineering/)
