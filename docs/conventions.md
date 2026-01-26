# Coding Conventions

> This document defines the coding standards for this project. AI agents and developers should follow these conventions.

## General Principles

1. **Clarity over cleverness**: Write code that is easy to understand
2. **Consistency**: Follow established patterns in the codebase
3. **Simplicity**: Prefer simple solutions over complex ones
4. **Documentation**: Document the "why", not the "what"

## Code Style

### Formatting

<!-- TODO: Customize for your project -->

- **Indentation**: 2 spaces (no tabs)
- **Line length**: Maximum 100 characters
- **Quotes**: Single quotes for strings
- **Semicolons**: [Required/Optional based on your preference]
- **Trailing commas**: Yes, in multiline structures

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `user-service.ts` |
| Classes | PascalCase | `UserService` |
| Functions | camelCase | `getUserById` |
| Variables | camelCase | `currentUser` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Interfaces | PascalCase with I prefix (optional) | `IUser` or `User` |
| Types | PascalCase | `UserResponse` |
| Enums | PascalCase | `UserStatus` |

### File Organization

```
src/
├── components/          # UI components
│   └── ComponentName/
│       ├── index.ts
│       ├── ComponentName.tsx
│       ├── ComponentName.test.tsx
│       └── ComponentName.styles.css
├── services/            # Business logic
├── utils/               # Helper functions
├── types/               # Type definitions
├── hooks/               # Custom hooks (React)
├── constants/           # Application constants
└── tests/               # Test utilities
```

## Functions

### Guidelines

- Keep functions small (< 30 lines ideally)
- Single responsibility principle
- Maximum 3-4 parameters
- Use destructuring for options objects

### Documentation

```typescript
/**
 * Brief description of what the function does.
 * 
 * @param userId - The unique identifier of the user
 * @param options - Additional options for the query
 * @returns The user object or null if not found
 * @throws {NotFoundError} When user doesn't exist and throwOnMissing is true
 * 
 * @example
 * const user = await getUser('123', { includeProfile: true });
 */
async function getUser(userId: string, options?: GetUserOptions): Promise<User | null> {
  // implementation
}
```

## Error Handling

### Pattern

```typescript
// DO: Use specific error types
throw new ValidationError('Email is required');

// DO: Handle errors at appropriate levels
try {
  await saveUser(user);
} catch (error) {
  if (error instanceof ValidationError) {
    return res.status(400).json({ error: error.message });
  }
  throw error; // Re-throw unexpected errors
}

// DON'T: Swallow errors silently
try {
  await saveUser(user);
} catch (error) {
  // Bad: error is ignored
}
```

## Testing

### Naming

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid input', async () => {});
    it('should throw ValidationError when email is missing', async () => {});
    it('should hash password before saving', async () => {});
  });
});
```

### Structure

- One assertion per test (when practical)
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Clean up test data

## Git Conventions

### Commit Messages

Format: `<type>(<scope>): <description>`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, no code change
- `refactor`: Code change without feature/fix
- `test`: Adding/updating tests
- `chore`: Maintenance tasks

Examples:
```
feat(auth): add password reset functionality
fix(api): handle null response from user service
docs(readme): update installation instructions
```

### Branch Naming

- `feature/short-description`
- `fix/issue-number-description`
- `docs/what-is-documented`
- `refactor/what-is-refactored`

## Code Review Checklist

Before submitting for review:

- [ ] Code follows project conventions
- [ ] Tests are written and passing
- [ ] No lint or type errors
- [ ] Documentation is updated
- [ ] Commit messages are clear
- [ ] PR description explains the changes

## Resources

- [Effective TypeScript](https://effectivetypescript.com/)
- [Clean Code principles](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
