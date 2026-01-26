---
applyTo: "**/*.md,**/docs/**"
description: "Documentation guidelines"
---

# Documentation Guidelines

## Writing Style

### Tone & Voice
- Use clear, concise language
- Write in active voice
- Be direct and helpful
- Avoid jargon unless necessary (define it when used)
- Use second person ("you") for instructions

### Structure
- Start with a brief overview
- Use headings to organize content
- Include table of contents for long documents
- End with next steps or related resources

## Markdown Best Practices

### Headings
- Use sentence case for headings
- Don't skip heading levels
- Keep headings descriptive but concise

### Code Blocks
- Always specify the language for syntax highlighting
- Include comments for complex code
- Show expected output when helpful

```typescript
// Example: Creating a new user
const user = await createUser({
  email: 'user@example.com',
  name: 'John Doe'
});
// Returns: { id: 'abc123', email: '...', name: '...' }
```

### Lists
- Use bullet points for unordered items
- Use numbered lists for sequential steps
- Keep list items parallel in structure

### Links
- Use descriptive link text (not "click here")
- Check links periodically for validity
- Use relative links for internal documentation

## Documentation Types

### README
- Project overview and purpose
- Quick start guide
- Installation instructions
- Basic usage examples
- Links to detailed documentation

### API Documentation
- Endpoint descriptions
- Request/response examples
- Authentication requirements
- Error codes and handling
- Rate limiting information

### Architecture Docs
- System overview diagrams
- Component descriptions
- Data flow explanations
- Technology decisions and rationale

### How-To Guides
- Clear problem statement
- Prerequisites
- Step-by-step instructions
- Troubleshooting section

## Maintenance

- Review documentation with code changes
- Date-stamp or version documentation
- Archive outdated documentation
- Encourage team contributions
