---
paths:
   - '**/*.ts'
   - '**/*.d.ts'
---

# TypeScript Standards

## Prohibitions

| Practice                   | Explanation                     |
| -------------------------- | ------------------------------- |
| Using `any`                | Use `unknown` instead           |
| Mixed exports of same identifier | `export` + `export type` |
| Non-null assertions `!` without justification | Use optional chaining `?.` instead |
| Using `type` for object types | Use `interface` instead        |
