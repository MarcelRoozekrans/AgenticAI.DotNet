---
description: "Architectural context for domain-driven design"
purpose: "Help agents focus on architectural patterns and consistency"
---

# Architecture Context

This file provides quick access to architectural patterns for context optimization.

## Clean Architecture Layers

### Domain Layer (Entities, Use Cases)
- **Location**: `src/domain/`
- **Purpose**: Pure business logic, framework-agnostic
- **Rules**:
  - No external dependencies
  - No framework imports
  - Contains domain entities and business rules

### Application Layer (Use Cases, Services)
- **Location**: `src/application/`
- **Purpose**: Orchestrate domain operations
- **Rules**:
  - Depends only on domain
  - Implements use case logic
  - No direct framework coupling

### Infrastructure Layer (Repositories, Services)
- **Location**: `src/infrastructure/`
- **Purpose**: External integrations and implementations
- **Rules**:
  - Implements domain interfaces
  - Handles database, APIs, external services
  - Depends on domain and application

### Presentation Layer (Controllers, Views)
- **Location**: `src/presentation/` or `src/api/`
- **Purpose**: Request handling and response formatting
- **Rules**:
  - Thin controllers - no business logic
  - Validates input at boundaries
  - Delegates to application layer

## Domain-Driven Design Patterns

### Aggregate Roots
- Clear boundary for consistency
- Root entity responsible for internal consistency
- Access child entities through root only

### Value Objects
- Immutable, defined by attributes
- No identity, only equivalence
- Encapsulate related values

### Domain Events
- Significant occurrences in the domain
- Used for integration between aggregates
- Example: `UserCreatedEvent`, `PaymentProcessedEvent`

## Testing Pyramid

```
       /\
      /  \
     /    \ E2E Tests
    /      \
   /--------\
  /          \
 /    API     \ Integration Tests
/              \
/----------------\
/                  \
/     Unit Tests     \
/                    \
```

- **Unit Tests**: Domain logic, isolated components (~70%)
- **Integration Tests**: Component interaction, database (~20%)
- **E2E Tests**: Full workflow, user scenarios (~10%)

## File Structure Template

```
src/
├── domain/
│   ├── entities/
│   │   └── [Entity].ts
│   ├── value-objects/
│   │   └── [ValueObject].ts
│   └── repositories/
│       └── [Repository].interface.ts
├── application/
│   ├── use-cases/
│   │   └── [UseCase].ts
│   └── services/
│       └── [Service].ts
├── infrastructure/
│   ├── repositories/
│   │   └── [Repository].implementation.ts
│   └── services/
│       └── [ExternalService].ts
├── presentation/
│   ├── controllers/
│   │   └── [Controller].ts
│   └── validators/
│       └── [Validator].ts
└── __tests__/
    ├── domain/
    ├── application/
    └── integration/
```

## Dependency Direction

Always enforce: **Outer layers depend on inner layers, never the reverse**

```
Presentation → Application → Domain
Infrastructure → Domain
```

Never:
- Domain importing from Presentation or Infrastructure
- Application importing from Presentation
