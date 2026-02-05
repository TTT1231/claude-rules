---
paths:
  - '**/*.{ts,js,vue}'
---

# Code Style

## Immutability (Core)
Always create new objects, prohibit direct modification:

```javascript
// ❌ Wrong: Direct modification
function updateUser(user, name) {
   user.name = name // Mutation occurred!
   return user
}

// ✅ Correct: Maintain immutability
function updateUser(user, name) {
   return {
      ...user,
      name
   }
}
```

## File Organization
Many small files > Few large files:
- High cohesion, low coupling
- Typically 200-400 lines, maximum 800
- Extract utility functions from large components
- Organize by feature/domain, not by type

## Error Handling
Must handle errors completely:

```typescript
try {
   const result = await riskyOperation()
   return result
} catch (error) {
   console.error('Operation failed:', error)
   throw new Error('Detailed and user-friendly error message')
}
```

## Input Validation
Must validate user input:

```typescript
import { z } from 'zod'

const schema = z.object({
   email: z.string().email(),
   age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## Code Quality Checklist
Confirm before completing work:
- [ ] Code is readable, naming is clear
- [ ] Functions are short (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Errors handled correctly
- [ ] No console.log statements
- [ ] No hardcoded values
- [ ] No direct mutation (use immutable patterns)
