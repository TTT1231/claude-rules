---
paths:
   - '**/*.ts'
   - '**/*.vue'
---

# 前端网络请求规范

## 适用范围

**前端项目**：Vue前端项目中的 API 请求

---

## 请求层规范

### 核心规则

| 规则类型 | 说明                                 |
| -------- | ------------------------------------ |
| **必须** | 可复用的请求逻辑放在 `src/api/` 目录 |
| **必须** | composables 调用 api 层而非直接请求  |
| **建议** | 组件中通过 api 层或 composables 请求 |
| **允许** | 组件中一次性简单请求（非推荐）       |

### 代码示例

```ts
// ⚠️ 允许但不推荐：组件中一次性简单请求
<script setup lang="ts">
import { request } from '@/utils/request'
const data = await request.get('/api/user')
</script>

// ✅ 推荐：通过 api 层（便于复用、测试、维护）
<script setup lang="ts">
import { getUserInfo } from '@/api/user'
const data = await getUserInfo()
</script>

// ❌ 禁止：composables 中直接请求（必须调用 api 层）
import { request } from '@/utils/request'
export function useUser() {
   request.get('/api/user').then(...)
}

// ✅ 正确：composables 调用 api 层
import { getUserInfo } from '@/api/user'
export function useUser() {
   getUserInfo().then(...)
}
```

---

## API 层结构

```ts
// src/api/user/types.ts
export interface IUser {
   id: string
   name: string
}

export interface GetUserParams {
   id: string
}

// src/api/user/index.ts
import { request } from '@/utils/request'
import type { IUser, GetUserParams } from './types'

export function getUserInfo(params: GetUserParams): Promise<IUser> {
   return request.get('/user/info', { params })
}
```

---

## 后端响应格式

后端 NestJS API 统一返回以下格式：

```ts
{
   code: number // 0 = 成功，非 0 = 失败
   data: T // 响应数据
   message: string // 提示信息
   timestamp: number
}
```

**前端处理示例**：

```ts
// 组件中判断 code 处理成功/失败
const response = await getUserInfo({ id: '123' })

if (response.code === 0) {
   const user = response.data // 成功，使用 data
} else {
   console.error(response.message) // 失败，显示错误信息
}
```

**axios 拦截器自动解包（可选）**：

```ts
// src/utils/request.ts
axios.interceptors.response.use(
   (response) => {
      const { data } = response
      if (data.code === 0) {
         return data.data // 自动解包，直接返回 data
      }
      return Promise.reject(new Error(data.message))
   },
   (error) => Promise.reject(error)
)

// API 函数直接返回数据类型
export function getUserInfo(params: GetUserParams): Promise<IUser> {
   return request.get('/user/info', { params })
}
```
