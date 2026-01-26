# Project Instructions

> These instructions apply globally to all interactions with GitHub Copilot in this repository.

## Project Overview

<!-- TODO: Describe your project -->
This project is a [describe your application type] built with [your tech stack].

## Tech Stack

<!-- TODO: Update with your actual tech stack -->
- **Language**: [e.g., TypeScript, Python, Go]
- **Framework**: [e.g., React, FastAPI, Express]
- **Database**: [e.g., PostgreSQL, MongoDB]
- **Testing**: [e.g., Jest, pytest, Go testing]

## Code Style & Conventions

### General Principles
- Write clean, readable, and maintainable code
- Follow the DRY (Don't Repeat Yourself) principle
- Prefer composition over inheritance
- Use meaningful variable and function names
- Add comments for complex logic, not obvious code

### Formatting
- Use consistent indentation (spaces/tabs as per project config)
- Keep lines under 100 characters when possible
- Use blank lines to separate logical blocks

### Naming Conventions
<!-- TODO: Customize for your language/framework -->
- **Files**: Use kebab-case for file names (e.g., `user-service.ts`)
- **Classes**: Use PascalCase (e.g., `UserService`)
- **Functions**: Use camelCase (e.g., `getUserById`)
- **Constants**: Use UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)
- **Variables**: Use camelCase (e.g., `currentUser`)

## Project Structure

<!-- TODO: Document your project structure -->
```
src/
├── components/     # UI components
├── services/       # Business logic
├── utils/          # Helper functions
├── types/          # Type definitions
└── tests/          # Test files
```

## Git Conventions

### Commit Messages
Use conventional commits format:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Code style changes (formatting, etc.)
- `refactor:` - Code refactoring
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks

### Branch Naming
- `feature/description` - New features
- `fix/description` - Bug fixes
- `docs/description` - Documentation updates

## Testing Requirements

- Write unit tests for all new functions
- Maintain minimum 80% code coverage
- Include edge cases and error scenarios
- Use descriptive test names

## Documentation

- Keep documentation minimal and focused
- Add comments only for complex or non-obvious logic
- Code should be self-documenting through clear naming and structure

## Security Guidelines

- Never commit secrets or API keys
- Validate all user inputs
- Use parameterized queries for database operations
- Follow principle of least privilege

## Error Handling

- Use try-catch blocks appropriately
- Provide meaningful error messages
- Log errors with sufficient context
- Handle edge cases gracefully
