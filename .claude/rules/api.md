# API 规范

## 响应格式

```typescript
interface ApiResponse<T> {
   code: number // HTTP 状态码
   data: T
   message: string
   timestamp: number
}
```

## 文件组织

| 项目类型 | API 文件位置         |
| -------- | -------------------- |
| Vue 单体 | `src/api/`           |
| Monorepo | `apps/[项目名]/api/` |

## URL 规则

| 规则        | 正确             | 错误            |
| ----------- | ---------------- | --------------- |
| 名词复数    | `/users`         | `/user`         |
| 小写+连字符 | `/user-profiles` | `/userProfiles` |
| 不超过 3 层 | `/users/1/posts` | `/a/b/c/d`      |
| 版本前缀    | `/api/v1/users`  | `/api/users`    |
