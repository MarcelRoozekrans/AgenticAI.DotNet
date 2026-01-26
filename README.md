# 🤖 Agentic AI Template

A template repository for building reliable AI workflows using **agentic primitives** and **context engineering**.

Based on the [GitHub Blog: How to build reliable AI workflows](https://github.blog/ai-and-ml/github-copilot/how-to-build-reliable-ai-workflows-with-agentic-primitives-and-context-engineering/).

## 🚀 Quick Start

1. **Use this template**: Click "Use this template" → "Create a new repository"
2. **Clone your new repo**: `git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git`
3. **Customize**: Edit the files in `.github/` to match your project's needs
4. **Start coding**: Open in VS Code with GitHub Copilot enabled

## 🎯 Setup from Scratch: Complete Guide

Follow these steps to fully configure your forked template for your project:

### Step 1: Initialize Your Project

```bash
cd your-repo
git checkout -b setup/initial-config
```

### Step 2: Customize Core Configuration Files

#### 2a. Update `apm.yml` (Priority: HIGH)
This is your project's DNA. Edit `.github/apm.yml`:

```yaml
# At the top, update project metadata
name: "YourProjectName"
description: "Your project description"
author: "Your Team"

# Update scripts for your tech stack
scripts:
  dev: "npm run dev"        # or: python app.py, go run main.go
  build: "npm run build"
  test: "npm run test"
  lint: "npm run lint"
  format: "npm run format"

# Update dependencies
dependencies:
  required:
    - node: ">= 18.0.0"     # Change based on your tech stack
  optional:
    - docker: ">= 20.0.0"
```

#### 2b. Update `copilot-instructions.md` (Priority: HIGH)
Edit `.github/copilot-instructions.md`:

```markdown
## Project Overview
[Replace with YOUR project description]

## Tech Stack
- **Language**: TypeScript / Python / Go / [your language]
- **Framework**: React / FastAPI / Express / [your framework]
- **Database**: PostgreSQL / MongoDB / [your database]
- **Testing**: Jest / pytest / [your test framework]

## Code Style & Conventions
[Update with your project's specific rules]

## Project Structure
[Document YOUR folder organization]
```

#### 2c. Update Domain Instructions
Edit files in `.github/instructions/`:
- **frontend.instructions.md**: Update `applyTo` patterns and framework guidelines
- **backend.instructions.md**: Add backend-specific patterns and database conventions
- **testing.instructions.md**: Define your testing strategy and coverage targets
- **docs.instructions.md**: Set documentation standards

Example for frontend.instructions.md:
```markdown
# Frontend Development Guidelines

This guide applies to: **/*.{js,jsx,ts,tsx,css,scss}

## Framework: React
- Use functional components with hooks
- Organize by feature
- [Add your specific patterns]
```

#### 2d. Customize Chat Modes
Edit files in `.github/chatmodes/`:

Update role definitions matching your team:
- **architect.chatmode.md**: For system design and planning
- **backend-dev.chatmode.md**: For your backend tech stack
- **frontend-dev.chatmode.md**: For your frontend framework
- **reviewer.chatmode.md**: For code quality focus

Example snippet:
```markdown
# Backend Developer Mode

You are an expert backend developer specializing in [Node.js/Python/Go].

## Your Responsibilities
- Implement APIs following [your architecture style]
- Write server-side tests using [your test framework]
- Follow database patterns: [your patterns]

## Tools You Have Access To
- codebase: Full repository access
- search: Code searching
- editFiles: File creation and modification
- runCommands: Test and build execution
- problems: Error diagnostics
```

### Step 3: Define Your Project Structure

Create or update these core files:

#### 3a. `docs/architecture.md`
Document your system architecture:
```markdown
# Architecture

## System Overview
[Diagrams and description]

## Layers
- Presentation Layer: [Your UI]
- Business Logic: [Your services]
- Data Access: [Your database layer]
- External Integrations: [APIs, services]

## Key Design Patterns
- [List patterns used in your project]
```

#### 3b. `docs/conventions.md`
Codify your development standards:
```markdown
# Coding Conventions

## Naming
- Files: kebab-case.ts
- Classes: PascalCase
- Functions: camelCase
- Constants: UPPER_SNAKE_CASE

## Error Handling
[Your error strategy]

## Async Patterns
[Your async/await or promise patterns]
```

#### 3c. `.memory.md`
Initialize agent memory:
```markdown
# Project Memory

## Project Context
- **Name**: Your Project
- **Purpose**: What it does
- **Tech Stack**: [Your stack]

## Important Patterns
- [Pattern 1]: Used in [modules]
- [Pattern 2]: Used in [modules]

## Failed Approaches (Don't Repeat)
- [What didn't work and why]

## Known Issues & Quirks
- [List any gotchas]

## Critical Success Factors
- [What makes this project succeed]

## External Dependencies
- [Critical third-party services]
```

### Step 4: Set Up Development Environment

#### 4a. Create your tech stack setup
Example for Node.js:
```bash
npm init -y
npm install [your dependencies]
npm install --save-dev typescript jest eslint prettier
```

#### 4b. Configure build/test scripts in `package.json`:
```json
{
  "scripts": {
    "dev": "your-dev-command",
    "build": "your-build-command",
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```

#### 4c. Verify apm.yml scripts match your setup:
```bash
npm run dev          # Should work
npm run build        # Should work
npm run test         # Should work
```

### Step 5: Create Your First Specification

Use `.github/instructions/docs.instructions.md` and create your first feature spec:

```bash
# Create: docs/specs/auth-system.spec.md
```

Use the template at `docs/specs/_template.spec.md`:
```markdown
# User Authentication System

## Problem Statement
Users need a secure way to register, login, and manage their sessions.

## Functional Requirements
1. User registration with email validation
2. Login with email/password
3. Password reset flow
4. Session management with JWT

## Technical Design
[Your architecture for this feature]

## Implementation Plan
1. Create User entity
2. Implement auth service
3. Create API endpoints
4. Add comprehensive tests
5. Security review

## Acceptance Criteria
- [ ] All unit tests pass (100% coverage for domain logic)
- [ ] API tests pass
- [ ] Security audit complete
- [ ] Documentation updated

## Risks & Mitigations
- **Risk**: Password security breaches
  **Mitigation**: Use bcrypt, OWASP guidelines
```

### Step 6: Test Your Agentic Setup

#### 6a. Open VS Code with GitHub Copilot
```bash
code .
```

Verify you have:
- ✅ GitHub Copilot Chat extension installed
- ✅ Your project open in VS Code
- ✅ Signed into GitHub

#### 6b. Test a simple architect request
Open Chat and try:
```
@workspace /chatmode architect

Design the folder structure for a user management system.
Reference docs/architecture.md for our existing patterns.
```

#### 6c. Test a developer request
```
@workspace /chatmode backend-dev

Implement a UserRepository following the patterns in architecture.md.
Use TDD: write tests first, then implement.
```

#### 6d. Test library documentation (Context7)
```
@dev How do I create a React hook for form validation?
```
(This uses the Context7 MCP server configured in apm.yml)

### Step 7: Configure Quality Gates

Update `apm.yml` quality-gates section with YOUR standards:

```yaml
quality-gates:
  phase1:
    - "Architecture diagram created"
    - "Task breakdown estimated"
    - "Team consensus on approach"
  
  phase2:
    - "Tests written first (Red)"
    - "Implementation complete (Green)"
    - "Code refactored (Blue)"
    - "All acceptance criteria passing"
    - "Code coverage >80%"
  
  phase3:
    - "Security review passed"
    - "Performance benchmarks acceptable"
    - "No SOLID violations detected"
  
  phase5:
    - "All tests pass"
    - "Build succeeds"
    - "No static analysis warnings"
    - "Production deployment verified"
```

### Step 8: Commit Your Setup

```bash
git add .github/ docs/ .memory.md apm.yml package.json
git commit -m "chore: initial agentic setup configuration"
git push origin setup/initial-config
```

Create a pull request and review all your customizations before merging to main.

---

## ✅ Checklist: Ready to Go

- [ ] `apm.yml` customized with your project name and scripts
- [ ] `copilot-instructions.md` updated with your tech stack
- [ ] Domain instructions in `.github/instructions/` match your languages
- [ ] Chat modes defined for your team roles
- [ ] `docs/architecture.md` documents your system
- [ ] `docs/conventions.md` codifies your standards
- [ ] `.memory.md` initialized with project knowledge
- [ ] First feature spec created and ready
- [ ] GitHub Copilot Chat installed and working
- [ ] First agentic workflow tested successfully

**You're now ready to start your first feature implementation!** 🚀

---

## 📁 Repository Structure

```
.github/
├── copilot-instructions.md          # Global repository rules
├── instructions/                    # Domain-specific instructions
│   ├── frontend.instructions.md     # Frontend development guidelines
│   ├── backend.instructions.md      # Backend development guidelines
│   ├── testing.instructions.md      # Testing guidelines
│   └── docs.instructions.md         # Documentation guidelines
├── chatmodes/                       # Role-based AI modes
│   ├── architect.chatmode.md        # Planning specialist
│   ├── frontend-dev.chatmode.md     # UI development specialist
│   ├── backend-dev.chatmode.md      # API/backend specialist
│   └── reviewer.chatmode.md         # Code review specialist
└── prompts/                         # Reusable agentic workflows
    ├── code-review.prompt.md        # Code review workflow
    ├── feature-implementation.prompt.md  # Feature implementation
    ├── bug-fix.prompt.md            # Bug fixing workflow
    └── refactor.prompt.md           # Refactoring workflow

docs/
├── specs/                           # Feature specifications
│   └── _template.spec.md            # Specification template
├── architecture.md                  # Project architecture
└── conventions.md                   # Coding conventions

.memory.md                           # Agent memory (project knowledge)
.context.md                          # Context helper file
```

## ⚙️ Agent Package Manager (apm.yml)

The `apm.yml` file is the **central configuration hub** for your agentic setup. It defines how your project behaves within AI-driven workflows.

### Key Sections

| Section | Purpose |
|---------|---------|
| `scripts` | Development commands (dev, build, test, lint, docs) |
| `dependencies` | Required/optional/dev dependencies and versions |
| `context-windows` | Token budgets and tool access for different roles (architect, developer, reviewer) |
| `mcp` | Model Context Protocol settings (context7, filesystem, shell, github) |
| `inner-loop` / `outer-loop` | Session modes for individual development vs. team orchestration |
| `quality-gates` | Phase-based quality criteria (specs, tests, security, performance) |
| `structure` | Project folder organization (src/, tests/, docs/, specs/) |
| `monitoring` | Observability configuration (error tracking, performance, logs) |

### Example: Using apm.yml

**Scripts** - Run any command from apm.yml:
```bash
npm run dev          # Fast development mode
npm run test:coverage # Check test coverage
npm run db:reset     # Reset database
```

**Context Windows** - Different roles get different AI focus:
- **Architect** (8K tokens): Planning & design
- **Developer** (12K tokens): Implementation & testing
- **Reviewer** (8K tokens): Code quality & security

**Quality Gates** - Automated checkpoints:
```
Phase 1 (Architect): Spec complete → Architecture diagrammed → Tasks estimated
Phase 2 (Developer): Tests written → Code green → Refactored → Criteria passing
Phase 3 (Reviewer): Security ✓ → Performance ✓ → Compliance ✓
Phase 5 (Verify): All tests pass → Build succeeds → Runtime verified
```

### How to Customize

1. **Update scripts** for your tech stack
2. **Adjust context windows** for your team's role structure
3. **Configure MCP servers** for the tools you need (context7 for library docs, github for repos, etc.)
4. **Set quality gates** matching your project standards
5. **Define dependencies** with version constraints

---

## 🧩 The Three-Layer Framework

### Layer 1: Markdown Prompt Engineering
Use structured Markdown to guide AI reasoning:
- **Context loading**: Links inject relevant information
- **Role activation**: "You are an expert [role]"
- **Validation gates**: Human checkpoints at critical decisions

### Layer 2: Agent Primitives
Reusable building blocks:

| File Type | Purpose |
|-----------|---------|
| `.instructions.md` | Domain-specific rules with `applyTo` patterns |
| `.chatmode.md` | Role-based expertise with tool boundaries |
| `.prompt.md` | Reusable workflows with validation |
| `.spec.md` | Implementation blueprints |
| `.memory.md` | Cross-session knowledge |
| `.context.md` | Information retrieval optimization |

### Layer 3: Context Engineering
Optimize AI focus:
- **Session splitting**: Separate sessions for different phases
- **Modular instructions**: Load only relevant context
- **Memory-driven development**: Preserve knowledge across sessions

## � Example: Starting the AI Programming Cycle

Here's how to write effective initial prompts that leverage the agentic primitives:

### Basic Feature Request
```
Implement a user authentication system with email/password login.

Follow the TDD approach - write tests first.
Use the existing patterns in this codebase.
```

### Detailed Specification-Driven Request
```
I need to implement a REST API for managing products.

## Context
- Review [architecture.md](docs/architecture.md) for system patterns
- Follow [backend.instructions.md](.github/instructions/backend.instructions.md)
- Use Clean Architecture with DDD patterns

## Requirements
1. CRUD operations for products (name, price, category, stock)
2. Pagination and filtering on list endpoint
3. Input validation with proper error responses
4. Unit tests for domain logic, integration tests for API

## Approach
1. First, create a specification in docs/specs/product-api.spec.md
2. Define the domain model (Product entity, value objects)
3. Use TDD: write failing tests → implement → refactor
4. Stop and review the plan before implementation

Do not modify any frontend code.
```

### Using Chat Modes
Switch to a specific mode for focused work:

1. **Planning phase** → Use `architect` mode:
   ```
   /chatmode architect
   
   Design the architecture for a notification system that supports 
   email, SMS, and push notifications. Consider scalability and 
   the ability to add new notification channels.
   ```

2. **Implementation phase** → Use `backend-dev` mode:
   ```
   /chatmode backend-dev
   
   Implement the NotificationService based on the architecture we designed.
   Follow TDD - start with the domain entities and use cases.
   ```

3. **Review phase** → Use `reviewer` mode:
   ```
   /chatmode reviewer
   
   Review the notification system implementation for security issues,
   SOLID principle violations, and test coverage gaps.
   ```

### Using Prompt Files
Trigger a complete workflow:
```
@workspace /prompt feature-implementation

Feature: Shopping cart functionality
- Add/remove items
- Calculate totals with tax
- Apply discount codes
```

### 🔄 Orchestrated Development Cycle

For larger features, use the automated development cycle that coordinates all roles:

```
@workspace /prompt development-cycle

Feature: User notification preferences
Context: Users should be able to configure how they receive notifications
Priority: High
```

This triggers a **5-phase automated cycle**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT CYCLE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Phase 1: 🏗️ ARCHITECT                                         │
│  ├─ Requirements analysis                                       │
│  ├─ Domain modeling (DDD)                                       │
│  ├─ Technical design                                            │
│  └─ Task breakdown                                              │
│           │                                                     │
│           ▼                                                     │
│  🚨 GATE 1: Review architecture before proceeding               │
│           │                                                     │
│           ▼                                                     │
│  Phase 2: 👨‍💻 DEVELOPER                                          │
│  ├─ TDD implementation (Red → Green → Refactor)                 │
│  ├─ Clean Architecture compliance                               │
│  └─ SOLID principles                                            │
│           │                                                     │
│           ▼                                                     │
│  🚨 GATE 2: Review implementation before proceeding             │
│           │                                                     │
│           ▼                                                     │
│  Phase 3: 🔍 REVIEWER                                           │
│  ├─ Code quality assessment                                     │
│  ├─ Security review                                             │
│  └─ SOLID/Clean Architecture compliance                         │
│           │                                                     │
│           ▼                                                     │
│  🚨 GATE 3: Approve or request changes                          │
│           │                                                     │
│     ┌─────┴─────┐                                               │
│     ▼           ▼                                               │
│  Phase 4:    Phase 5:                                           │
│  🔧 ITERATE  ✅ COMPLETE                                        │
│  (if issues)  (if approved)                                     │
│     │                                                           │
│     └──────► Back to Gate 2                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**For smaller tasks**, use the quick cycle:
```
@workspace /prompt quick-task

Task: Fix null pointer in user service
Type: bugfix
Scope: user-service
```

### Key Tips for Effective Prompts
- ✅ Reference existing docs and patterns with links
- ✅ Specify the development approach (TDD, DDD, etc.)
- ✅ Include validation gates ("stop and review before...")
- ✅ Set clear boundaries ("do not modify...")
- ✅ Break complex features into phases
- ❌ Don't be vague ("make it better")
- ❌ Don't skip context ("you know what I mean")

---

## �📝 How to Customize

### 1. Global Instructions
Edit `.github/copilot-instructions.md` with your:
- Project overview and tech stack
- Coding standards and conventions
- Commit message format
- File organization rules

### 2. Domain Instructions
Customize files in `.github/instructions/`:
- Adjust `applyTo` patterns for your file types
- Add framework-specific guidelines
- Include project-specific patterns

### 3. Chat Modes
Modify files in `.github/chatmodes/`:
- Define roles matching your team structure
- Set appropriate MCP tool boundaries
- Configure model preferences

### 4. Prompts
Create workflows in `.github/prompts/`:
- Build step-by-step processes
- Add validation checkpoints
- Include context loading phases

## 🔧 Requirements

- [VS Code](https://code.visualstudio.com/)
- [GitHub Copilot](https://github.com/features/copilot) subscription
- GitHub Copilot Chat extension

## 📚 Resources

- [GitHub Blog: Agentic Primitives Guide](https://github.blog/ai-and-ml/github-copilot/how-to-build-reliable-ai-workflows-with-agentic-primitives-and-context-engineering/)
- [VS Code Copilot Customization](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [Awesome AI-Native](https://danielmeppiel.github.io/awesome-ai-native/)
- [Spec-Kit](https://github.com/github/spec-kit) - Specification-driven development
- [APM](https://github.com/danielmeppiel/apm) - Agent Package Manager

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

---

⭐ **Star this repo** if you find it useful!  
🍴 **Fork it** to create your own agentic workflows!
