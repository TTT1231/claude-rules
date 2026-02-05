# API Standards

## Response Format

```typescript
interface ApiResponse<T> {
   code: number // HTTP status code
   data: T
   message: string
   timestamp: number
}
```

## File Organization

| Project Type | API File Location        |
| ------------ | ------------------------ |
| Vue Monolith | `src/api/`               |
| Monorepo     | `apps/[project-name]/api/` |

## URL Rules

| Rule           | Correct          | Incorrect        |
| -------------- | ---------------- | ---------------- |
| Plural nouns   | `/users`         | `/user`          |
| Lowercase + hyphen | `/user-profiles` | `/userProfiles` |
| Max 3 levels   | `/users/1/posts` | `/a/b/c/d`       |
| Version prefix | `/api/v1/users`  | `/api/users`     |
