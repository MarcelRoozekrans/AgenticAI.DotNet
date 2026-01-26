# Project Architecture

> Document your project's architecture here. This file helps AI agents understand your system structure.

## Overview

<!-- TODO: Add a high-level description of your project -->

Brief description of what this project does and its main components.

## System Diagram

<!-- TODO: Add or describe your architecture diagram -->

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Web App   │  │ Mobile App  │  │   CLI       │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        API Layer                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   API Gateway                        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Service A  │  │  Service B  │  │  Service C  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Database   │  │    Cache    │  │   Storage   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## Components

### Component 1: [Name]

**Purpose**: What this component does

**Technology**: Tech stack used

**Key Files**:
- `src/component1/` - Main implementation
- `src/component1/types.ts` - Type definitions

### Component 2: [Name]

**Purpose**: What this component does

**Technology**: Tech stack used

**Key Files**:
- `src/component2/` - Main implementation

## Data Flow

Describe how data flows through the system:

1. User action triggers...
2. Request is processed by...
3. Data is stored in...
4. Response is returned...

## Key Design Decisions

### Decision 1: [Title]

**Context**: Why this decision was needed

**Decision**: What was decided

**Rationale**: Why this approach was chosen

**Trade-offs**: What we gave up

### Decision 2: [Title]

**Context**: Why this decision was needed

**Decision**: What was decided

**Rationale**: Why this approach was chosen

## External Dependencies

| Dependency | Purpose | Documentation |
|------------|---------|---------------|
| Service A | Authentication | [Link] |
| Service B | Payments | [Link] |

## Development Setup

Refer to [README.md](../README.md) for setup instructions.

## Related Documentation

- [Conventions](./conventions.md)
- [API Documentation](./api.md)
