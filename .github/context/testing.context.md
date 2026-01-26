---
description: "Testing patterns and best practices"
purpose: "Ensure consistent, high-quality test implementation"
---

# Testing Context

Quick reference for testing patterns and conventions.

## Test Structure (AAA Pattern)

Every test follows Arrange → Act → Assert:

```typescript
describe('UserEntity', () => {
  it('should create user with valid email', () => {
    // Arrange - Set up test data and dependencies
    const email = 'user@example.com';
    
    // Act - Execute the code being tested
    const user = new UserEntity(email);
    
    // Assert - Verify the results
    expect(user.email).toBe(email);
    expect(user.createdAt).toBeDefined();
  });
});
```

## Test File Locations

```
src/
├── domain/
│   └── __tests__/
│       └── [Entity].test.ts
├── application/
│   └── __tests__/
│       └── [UseCase].test.ts
└── infrastructure/
    └── __tests__/
        └── [Repository].test.ts
```

## Test Coverage Requirements

- **Overall**: Minimum 80% coverage
- **Domain Logic**: 100% coverage (critical business rules)
- **Use Cases**: 90%+ coverage (orchestration logic)
- **Infrastructure**: 70%+ coverage (can exclude trivial getters/setters)

## Testing Levels

### Unit Tests
- **Focus**: Single component in isolation
- **Mocks**: Dependencies are mocked
- **Speed**: Fast (< 100ms per test)
- **Location**: Same directory as code, `__tests__/` folder

```typescript
it('should validate email format', () => {
  expect(() => new Email('invalid')).toThrow(InvalidEmailError);
  expect(() => new Email('valid@example.com')).not.toThrow();
});
```

### Integration Tests
- **Focus**: Multiple components working together
- **Mocks**: Minimal, use real implementations when possible
- **Speed**: Medium (< 1s per test)
- **Location**: `src/__tests__/integration/`

```typescript
it('should persist user to database and retrieve it', async () => {
  const repository = new UserRepository(db);
  const user = new User('john@example.com');
  
  await repository.save(user);
  const retrieved = await repository.findById(user.id);
  
  expect(retrieved).toEqual(user);
});
```

### Edge Cases to Always Test

1. **Happy Path**: Normal, expected behavior
2. **Error Cases**: Invalid inputs, exceptions
3. **Boundary Cases**: Empty, null, minimum, maximum values
4. **Concurrent Access**: Race conditions if applicable
5. **State Changes**: Before/after state verification

```typescript
describe('PaymentService', () => {
  it('should process valid payment', async () => { /* ... */ });
  it('should reject payment with insufficient funds', async () => { /* ... */ });
  it('should handle $0 payment', async () => { /* ... */ });
  it('should handle concurrent payments', async () => { /* ... */ });
  it('should update user balance after payment', async () => { /* ... */ });
});
```

## Test Naming Convention

Use descriptive names that explain behavior:

✅ **Good**:
- `should_create_user_with_valid_email`
- `should_throw_error_when_email_is_invalid`
- `should_persist_user_to_database`

❌ **Bad**:
- `test_user`
- `it works`
- `should handle stuff`

## Common Testing Utilities

### Test Fixtures (Setup Helpers)

```typescript
export const createTestUser = (overrides?: Partial<User>): User => {
  return new User({
    email: 'test@example.com',
    name: 'Test User',
    ...overrides,
  });
};
```

### Mock Factories

```typescript
export const mockUserRepository = (): jest.Mocked<UserRepository> => {
  return {
    save: jest.fn(),
    findById: jest.fn(),
    delete: jest.fn(),
  };
};
```

## Running Tests

```bash
# Run all tests
npm test

# Run specific file
npm test -- UserEntity.test.ts

# Watch mode
npm test -- --watch

# Coverage report
npm test -- --coverage
```

## Performance Considerations

- **Avoid real HTTP calls**: Use mocks or test fixtures
- **Avoid real database**: Use in-memory or test database
- **Avoid real file I/O**: Mock file system
- **Parallelize tests**: Jest runs tests in parallel by default
- **Use test seeds**: Consistent test data for reproducibility

## Assertions Cheat Sheet

```typescript
expect(value).toBe(expected);                    // Exact equality
expect(value).toEqual(expected);                 // Deep equality
expect(value).toThrow(ErrorType);                // Exception thrown
expect(fn).toHaveBeenCalled();                   // Function called
expect(fn).toHaveBeenCalledWith(args);           // Called with args
expect(array).toContain(item);                   // Array contains item
expect(value).toBeDefined();                     // Not undefined
expect(value).toBeNull();                        // Null check
expect(promise).resolves.toEqual(value);         // Promise resolves
expect(promise).rejects.toThrow(ErrorType);      // Promise rejects
```
