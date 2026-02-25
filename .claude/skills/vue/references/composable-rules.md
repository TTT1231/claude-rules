# Composable 编写规则

## 核心规则

### 单一职责

```typescript
// ✅ 单一职责
export function useTableData() {
   /* 只处理数据 */
}
export function useTableFilters() {
   /* 只处理筛选 */
}

// ❌ 职责混乱
export function useTable() {
   /* 数据 + 筛选 + 分页 + UI 逻辑混在一起 */
}
```

### API 层调用

```typescript
// ✅ 调用 API 层，不直接使用 axios
import { getUserList } from '@/api/user'

export function useUserList() {
   const data = ref<User[]>([])
   const loading = ref(false)
   const error = ref<Error | null>(null)

   const fetch = async () => {
      loading.value = true
      try {
         data.value = await getUserList()
      } catch (e) {
         error.value = e as Error
      } finally {
         loading.value = false
      }
   }

   return { data, loading, error, fetch }
}
```

### 响应式返回

```typescript
import { ref, reactive } from 'vue'

// ✅ 正确返回响应式数据
return { data, loading } // ref 直接返回
return state // reactive 整体返回

// ❌ 错误：解构 reactive 失去响应性
return { ...state }
```

---

## 类型导出

```typescript
export interface UseTableDataOptions {
   url: string
   immediate?: boolean
}

export interface UseTableDataReturn {
   data: Ref<DataItem[]>
   loading: Ref<boolean>
   fetch: () => Promise<void>
}

export function useTableData(options: UseTableDataOptions): UseTableDataReturn
```

---

## 命名规范

- 文件名：`use` 前缀 + 驼峰命名（如 `useTableData.ts`）
- 函数名：与文件名一致
- 返回类型：`Use{FunctionName}Return`
- 参数类型：`Use{FunctionName}Options`

---

## 错误处理

```typescript
// ✅ 完整的错误处理
export function useApiCall() {
   const loading = ref(false)
   const error = ref<Error | null>(null)

   const execute = async () => {
      loading.value = true
      error.value = null
      try {
         // ...
      } catch (e) {
         error.value = e instanceof Error ? e : new Error(String(e))
      } finally {
         loading.value = false
      }
   }

   return { loading, error, execute }
}
```

---

## 副作用清理 (Side-effect Cleanup)

在 Composable 中注册的事件监听器、定时器等副作用，必须在组件卸载时清理。

```typescript
import { onMounted, onUnmounted } from 'vue'

export function useWindowResize() {
   const width = ref(window.innerWidth)
   const height = ref(window.innerHeight)

   const handleResize = () => {
      width.value = window.innerWidth
      height.value = window.innerHeight
   }

   onMounted(() => {
      window.addEventListener('resize', handleResize)
   })

   // ✅ 必须清理副作用
   onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
   })

   return { width, height }
}
```

---

## 状态共享 (State Sharing)

Composable 默认每次调用都会创建新的状态。如果需要在多个组件间共享状态，可以将状态定义在函数外部。

```typescript
import { ref } from 'vue'

// ✅ 状态定义在外部，实现全局共享（类似简易 Pinia）
const sharedState = ref(0)

export function useSharedCounter() {
   const increment = () => {
      sharedState.value++
   }

   return { count: sharedState, increment }
}
```

> **注意**：对于复杂的全局状态管理，优先使用 Pinia。这种模式仅适用于简单的状态共享。
