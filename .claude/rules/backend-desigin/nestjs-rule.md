---
paths:
   - '**/*.controller.ts'
---

# Nestjs API规范

## URL 规则

| 规则        | 正确             | 错误            |
| ----------- | ---------------- | --------------- |
| 名词复数    | `/users`         | `/user`         |
| 小写+连字符 | `/user-profiles` | `/userProfiles` |
| 不超过 3 层 | `/users/1/posts` | `/a/b/c/d`      |
| 版本控制    | `/api/v1/users`  | `/api/users`    |

---

## 响应格式（统一）

```ts
interface ApiResponse<T = any> {
   code: number // 0 = 成功
   data: T
   message: string
   timestamp: number
}
```

---
