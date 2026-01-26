---
applyTo: "**/test/**,**/*.test.*,**/*.spec.*,**/__tests__/**,**/tests/**,**/.tests/**,**/testing/**,**/*.e2e.*,**/integration/**,**/fixtures/**,**/mocks/**"
description: "Testing guidelines and best practices"
---

# Testing Guidelines

## Test Structure

### Naming Convention
Use descriptive test names that explain what is being tested:
```
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid input', () => {});
    it('should throw error when email is invalid', () => {});
    it('should hash password before saving', () => {});
  });
});
```

### AAA Pattern
Follow Arrange-Act-Assert pattern:
```
// Arrange - Set up test data and conditions
const user = createTestUser();

// Act - Execute the code being tested
const result = await userService.createUser(user);

// Assert - Verify the expected outcome
expect(result.id).toBeDefined();
```

## Test Types

### Unit Tests
- Test individual functions/methods in isolation
- Mock all external dependencies
- Fast execution (milliseconds)
- High coverage of business logic

### Integration Tests
- Test component interactions
- Use test databases or containers
- Verify API contracts
- Test database operations

### End-to-End Tests
- Test complete user flows
- Run against staging environment
- Verify critical business paths
- Keep count minimal (expensive to maintain)
- **When applicable**: Feature that involves multiple components interacting
- **Examples**:
  - User signup → email confirmation → login → access dashboard
  - Create order → payment → confirmation email → inventory update
  - Upload file → processing → notification → download

## Best Practices

### Do's
- Test behavior, not implementation
- Keep tests independent (no shared state)
- Use descriptive assertion messages
- Clean up test data after each test
- Use factories for test data creation

### Don'ts
- Don't test framework/library code
- Don't use production data in tests
- Don't write flaky tests
- Don't skip tests without documentation
- Don't test private methods directly

## Mocking Guidelines

- Mock at the boundary (API, database, external services)
- Verify mock interactions when relevant
- Reset mocks between tests
- Use type-safe mocks when possible

## Coverage Goals

- Aim for 80%+ line coverage
- Focus on critical business logic
- Don't chase 100% (diminishing returns)
- Cover error paths and edge cases

## Test Data

### Factories
Create reusable test data factories:
```
const createTestUser = (overrides = {}) => ({
  id: 'test-id',
  email: 'test@example.com',
  name: 'Test User',
  ...overrides
});
```

### Fixtures
- Use for complex, static test data
- Keep in separate files
- Version control with tests
