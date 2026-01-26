---
applyTo: "**/*.{js,jsx,ts,tsx,css,scss,html,vue,svelte}"
description: "Frontend development guidelines"
---

# Frontend Development Guidelines

## Component Architecture

### Component Structure
- Use functional components with hooks
- Keep components small and focused (single responsibility)
- Extract reusable logic into custom hooks
- Separate presentation from business logic

### File Organization
```
components/
├── ComponentName/
│   ├── index.ts              # Export barrel
│   ├── ComponentName.tsx     # Main component
│   ├── ComponentName.test.tsx # Tests
│   ├── ComponentName.styles.css # Styles
│   └── ComponentName.types.ts # Type definitions
```

## Styling Guidelines

### CSS Best Practices
- Use CSS modules or styled-components for scoping
- Follow BEM naming convention for class names
- Use CSS variables for theming
- Mobile-first responsive design
- Avoid inline styles except for dynamic values

### Accessibility
- Use semantic HTML elements
- Include ARIA labels where needed
- Ensure keyboard navigation works
- Maintain color contrast ratios (WCAG 2.1)
- Add alt text to all images

## State Management

- Use local state for component-specific data
- Use context for shared state across component trees
- Consider state management libraries for complex global state
- Keep state as close to where it's used as possible

## Performance

- Implement lazy loading for routes and heavy components
- Use `useMemo` and `useCallback` appropriately
- Optimize images and assets
- Avoid unnecessary re-renders
- Use virtualization for long lists

## Testing

- Write unit tests for utility functions
- Write component tests for user interactions
- Use testing-library best practices
- Test accessibility with axe or similar tools

## Error Handling

- Implement error boundaries for graceful failures
- Show user-friendly error messages
- Log errors for debugging
- Handle loading states appropriately
