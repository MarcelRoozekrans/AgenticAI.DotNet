---
applyTo: "**/*.{py,go,java,cs,rb,rs,php}"
description: "Backend development guidelines"
---

# Backend Development Guidelines

## API Design

### RESTful Principles
- Use proper HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Return appropriate status codes
- Use plural nouns for resource endpoints
- Version your APIs (e.g., `/api/v1/`)
- Implement pagination for list endpoints

### Response Format
```json
{
  "data": {},
  "meta": {
    "timestamp": "ISO8601",
    "requestId": "uuid"
  },
  "errors": []
}
```

## Database Operations

### Query Best Practices
- Use parameterized queries (prevent SQL injection)
- Implement proper indexing
- Use transactions for multi-step operations
- Optimize N+1 query problems
- Use connection pooling

### Data Validation
- Validate all inputs at the API boundary
- Use schema validation libraries
- Sanitize data before storage
- Implement rate limiting

## Authentication & Authorization

- Use industry-standard protocols (OAuth2, JWT)
- Implement proper token refresh mechanisms
- Use secure password hashing (bcrypt, argon2)
- Apply principle of least privilege
- Log authentication events

## Error Handling

### Error Response Format
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": {}
  }
}
```

### Logging
- Log all errors with stack traces
- Include request context (user, request ID)
- Use structured logging (JSON format)
- Don't log sensitive data (passwords, tokens)

## Performance

- Implement caching strategies (Redis, in-memory)
- Use async operations where appropriate
- Optimize database queries
- Implement connection pooling
- Monitor and set up alerts

## Testing

- Write unit tests for business logic
- Write integration tests for API endpoints
- Use mocking for external dependencies
- Test error scenarios and edge cases
- Maintain test data factories
