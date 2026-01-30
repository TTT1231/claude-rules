---
paths:
   - '**/*.vue'
---

# Vue 规范

## 文件模板（唯一标准）

```vue
<script setup lang="ts">
// 组件名优先使用文件名，特殊情况（如递归组件）需要 defineOptions
</script>

<template>
   <!-- 内容 -->
</template>

<style lang="scss" scoped>
/* 样式 */
</style>
```

**禁止**：

- `<script setup>` 不加 `lang="ts"`（TS 项目）
- `<style>` 不加 `scoped`
- 使用 `CommonJS`规范（必须用 `ES Module`规范）
- 内联样式（非动态属性）

```vue
<!-- ❌ 禁止：静态内联样式 -->
<div style="color: red; margin: 10px;"></div>

<!-- ✅ 允许：动态内联样式 -->
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<div :style="{ opacity: isVisible ? 1 : 0 }"></div>
```

---

## 命名规范

| 类型     | 规范           | 示例                   |
| -------- | -------------- | ---------------------- |
| 组件文件 | PascalCase.vue | `UserProfile.vue`      |
| 文件夹   | kebab-case     | `user-profile/`        |
| 组件引用 | PascalCase     | `import User from ...` |

---

## Props 定义规范

### 选择标准

| 场景                        | 推荐方式                         |
| --------------------------- | -------------------------------- |
| Props 简单（≤3 个）         | `defineProps<{ }>` 内联          |
| Props 复杂（>3 个）或需复用 | `interface + withDefaults`       |
| 需要默认值                  | `withDefaults`（推荐 interface） |
| Props 类型需导出供外部使用  | `interface`（必须）              |

### 代码示例

```vue
<script setup lang="ts">
// ✅ 推荐：简单 Props 内联（≤3 个，无默认值）
defineProps<{
   title: string
   count?: number
}>()

// ✅ 推荐：复杂 Props + 默认值
interface Props {
   title: string
   items: string[]
   disabled?: boolean
   size?: 'small' | 'medium' | 'large'
}
withDefaults(defineProps<Props>(), {
   disabled: false,
   size: 'medium'
})

// ✅ 允许：简单 Props 也用 interface（如团队偏好统一）
interface Props {
   title: string
   count?: number
}
defineProps<Props>()

// ❌ 避免：内联 + withDefaults（应提取 interface）
withDefaults(
   defineProps<{
      title: string
      items: string[]
      disabled?: boolean
   }>(),
   { disabled: false }
)
</script>
```

### Props 类型导出

当 Props 类型需要被外部使用（如父组件传递类型、测试文件引用）时，必须导出：

```vue
<!-- UserProfile.vue -->
<script setup lang="ts">
// ✅ 导出 Props 类型供外部使用
export interface UserProfileProps {
   userId: string
   showAvatar?: boolean
}

defineProps<UserProfileProps>()
</script>
```

```ts
// 父组件中复用类型
import type { UserProfileProps } from './UserProfile.vue'

const props: UserProfileProps = {
   userId: '123',
   showAvatar: true
}
```

### 命名规范

| 场景      | 规范       | 示例                           |
| --------- | ---------- | ------------------------------ |
| Prop 定义 | camelCase  | `itemCount: number`            |
| Prop 传递 | kebab-case | `<UserCard :item-count="5" />` |
| v-model   | modelValue | `<input v-model="value" />`    |

---

## Emits 规范

### 事件命名

| 类型         | 规范            | 示例                        |
| ------------ | --------------- | --------------------------- |
| 更新类事件   | `update:xxx`    | `update:modelValue`         |
| 动作类事件   | 动词原形/过去式 | `submit` / `submitted`      |
| 事件参数传递 | kebab-case      | `@item-click="handleClick"` |

### 代码示例

```vue
<!-- 示例 1：完整类型定义（推荐） -->
<script setup lang="ts">
interface Item {
   id: string
   name: string
}

interface Emits {
   (e: 'update:modelValue', value: string): void
   (e: 'submit', data: FormData): void
   (e: 'item-click', item: Item, index: number): void
}

const emit = defineEmits<Emits>()

defineProps<{
   modelValue: string
   items: Item[]
}>()

const formData = new FormData()
</script>

<template>
   <!-- ✅ v-model 自动映射到 update:modelValue -->
   <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)" />

   <!-- ✅ 动作事件 -->
   <button @click="emit('submit', formData)">提交</button>

   <!-- ✅ 事件参数（注意：key 应使用 item.id 而非 index） -->
   <div v-for="(item, index) in items" :key="item.id" @click="emit('item-click', item, index)">
      {{ item.name }}
   </div>
</template>
```

```vue
<!-- 示例 2：简单场景内联（≤3 个事件） -->
<script setup lang="ts">
const emit = defineEmits<{
   'update:modelValue': [value: string]
   submit: [data: FormData]
}>()
</script>
```

```ts
// ❌ 避免：无类型约束
const emit = defineEmits(['update:modelValue', 'submit'])
```

### 常用事件约定

| 用途     | 事件名              | 参数        |
| -------- | ------------------- | ----------- |
| 双向绑定 | `update:modelValue` | 新值        |
| 确认操作 | `confirm`           | 数据        |
| 取消操作 | `cancel`            | -           |
| 删除操作 | `delete` / `remove` | id / 数据   |
| 点击项   | `item-click`        | item, index |
| 状态变化 | `change`            | 新值        |
| 加载完成 | `loaded`            | 数据        |

---

## 响应式变量规范（ref vs reactive）

### 选择标准

| 场景                       | 推荐方式        | 理由                          |
| -------------------------- | --------------- | ----------------------------- |
| 基本类型（string、number） | `ref`           | reactive 无法包装基本类型     |
| 对象（需要整体替换）       | `ref`           | ref.value = newObj 可整体替换 |
| 对象（属性解构时）         | `ref` 或 toRefs | reactive 解构会失去响应性     |
| 对象（保持引用，访问频繁） | `reactive`      | 避免 `.value`，代码更简洁     |
| 数组（需要整体替换）       | `ref`           | ref.value = [] 可整体替换     |
| 从 setup 返回的状态        | `reactive`      | 配合 `toRefs` 解构时使用      |

> **默认原则**：优先使用 `ref`，只有在对象需要频繁访问属性且不涉及整体替换时使用 `reactive`。

### 代码示例

```vue
<script setup lang="ts">
// ✅ 推荐：基本类型用 ref
const count = ref(0)
const message = ref('')

// ✅ 推荐：对象用 ref（可能整体替换）
const user = ref<User | null>(null)
user.value = await fetchUser() // 整体替换

// ✅ 推荐：数组用 ref（需要整体替换）
const list = ref<Item[]>([])
list.value = await fetchList() // 整体替换
list.value.push(newItem) // 也能修改属性

// ✅ 允许：reactive 用于表单（访问频繁，不整体替换）
const form = reactive({
   username: '',
   password: '',
   email: ''
})
form.username = 'xxx' // 无需 .value

// ❌ 避免：reactive 解构（失去响应性）
const state = reactive({ count: 0, name: '' })
const { count, name } = state // count、name 不再响应

// ✅ 正确：使用 toRefs 保持响应性
const state = reactive({ count: 0, name: '' })
const { count, name } = toRefs(state)

// ❌ 避免：reactive 包装 null
const user = reactive<User | null>(null) // 报错：无法包装 null

// ✅ 正确：ref 可包装 null
const user = ref<User | null>(null)
</script>
```

### Composable 返回值规范

```ts
// ✅ 推荐：返回 ref 对象（可直接解构）
export function useCounter() {
   const count = ref(0)
   const double = computed(() => count.value * 2)

   function increment() {
      count.value++
   }

   return { count, double, increment }
}

// 使用
const { count, double, increment } = useCounter()

// ✅ 允许：返回 reactive（配合 toRefs）
export function useForm() {
   const form = reactive({ username: '', password: '' })

   function reset() {
      form.username = ''
      form.password = ''
   }

   return { ...toRefs(form), reset }
}

// 使用
const { username, password, reset } = useForm()
```

---

## Composables 规范

### 命名规范

| 类型   | 规范      | 示例               |
| ------ | --------- | ------------------ |
| 文件名 | useXxx.ts | `useUser.ts`       |
| 函数名 | useXxx    | `useUser()`        |
| 返回值 | ref 对象  | `{ count, reset }` |

### 组织规则

| 规则         | 说明                                     |
| ------------ | ---------------------------------------- |
| **单一职责** | 每个 composable 只处理一个关注点         |
| **参数类型** | 必须 export interface 供外部使用         |
| **返回值**   | 优先返回 ref，避免返回 reactive          |
| **API 调用** | 必须调用 api 层，禁止直接 request        |
| **存放位置** | `src/composables/` 或组件同目录 `hooks/` |

### 代码示例

```ts
// src/composables/useUser.ts
import { ref } from 'vue'
import { getUserInfo } from '@/api/user'

// ✅ 导出参数类型供外部使用
export interface UseUserParams {
   userId: string
   autoFetch?: boolean
}

// ✅ 导出返回值类型
export interface UseUserReturn {
   user: Ref<User | null>
   loading: Ref<boolean>
   error: Ref<Error | null>
   fetch: () => Promise<void>
}

export function useUser(params: UseUserParams): UseUserReturn {
   const user = ref<User | null>(null)
   const loading = ref(false)
   const error = ref<Error | null>(null)

   async function fetch() {
      loading.value = true
      error.value = null
      try {
         user.value = await getUserInfo({ id: params.userId })
      } catch (e) {
         error.value = e as Error
      } finally {
         loading.value = false
      }
   }

   if (params.autoFetch) {
      fetch()
   }

   return { user, loading, error, fetch }
}
```

```vue
<!-- 使用 -->
<script setup lang="ts">
import type { UseUserParams } from '@/composables/useUser'

const params: UseUserParams = {
   userId: '123',
   autoFetch: true
}
const { user, loading, fetch } = useUser(params)
</script>
```

### 禁止模式

```ts
// ❌ 禁止：composable 中直接请求
export function useUser() {
   import { request } from '@/utils/request'
   request.get('/api/user') // 必须调用 api 层
}

// ❌ 禁止：返回 reactive（解构时失去响应性）
export function useCounter() {
   const state = reactive({ count: 0 })
   return state // ❌
}

// ✅ 正确：返回 ref 或 toRefs
export function useCounter() {
   const count = ref(0)
   return { count }
}
```

---

## 多文件组件

### 拆分标准

**唯一标准：复杂度**

| 条件         | 说明                     | 示例                         |
| ------------ | ------------------------ | ---------------------------- |
| **多职责**   | 组件承担多个不相关的功能 | 同时处理数据展示、编辑、导出 |
| **深层嵌套** | 模板嵌套层级 > 4 层      | div 里套 div 再套 div...     |
| **复用需求** | 子逻辑可能被其他地方复用 | 弹窗、表单项、列表项         |
| **类型导出** | 需要导出类型供外部使用   | 复杂的 Props/Emits 类型      |

> **行数不是拆分依据**：一个 200 行但逻辑复杂的组件应该拆分，而一个 600 行但结构简单的列表组件可以保留。

### 单文件 vs 多文件选择

| 场景                                               | 推荐方式 | 目录结构                               | 导入方式                                           |
| -------------------------------------------------- | -------- | -------------------------------------- | -------------------------------------------------- |
| 简单组件（单一职责、浅层嵌套、无复用子组件）       | 单文件   | `components/UserCard.vue`              | `import UserCard from '@/components/UserCard.vue'` |
| 复杂组件（多职责、深层嵌套、有子组件、需导出类型） | 多文件   | `components/Menu/src/...` + `index.ts` | `import { Menu } from '@/components/Menu'`         |

### 目录结构（使用 Barrel Pattern）

```
Menu/
├── src/
│   ├── Menu.vue           # 主组件（必须默认导出）
│   ├── MenuItem.vue       # 子组件
│   ├── types.ts           # 类型定义
│   └── constants.ts       # 常量
└── index.ts               # 桶文件
```

```ts
// Menu/index.ts
export { default as Menu } from './src/Menu.vue'
export { default as MenuItem } from './src/MenuItem.vue'
export * from './src/types'
export * from './src/constants'

// 使用
import { Menu, MenuItem, IMenu } from '@/components/Menu'

// ❌ 避免：深层导入
import Menu from '@/components/Menu/src/Menu.vue'
```

**说明**：单文件组件允许直接导入，多文件组件（有 `src/` 目录）应通过桶文件导入。

---

## 前端 API 请求规范

### 核心规则

| 规则类型 | 说明                                 |
| -------- | ------------------------------------ |
| **必须** | 可复用的请求逻辑放在 `src/api/` 目录 |
| **必须** | composables 调用 api 层而非直接请求  |
| **建议** | 组件中通过 api 层或 composables 请求 |
| **允许** | 组件中一次性简单请求（非推荐）       |

> **决策依据**：简单的一次性请求可以直接写在组件中，但任何可能复用、包含复杂逻辑（重试、缓存、转换）的请求必须放在 api 层。

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

### API 层示例

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
