# 组件分类与拆分规则

## 组件分类

| 类型       | 特征                          | 示例                            |
| ---------- | ----------------------------- | ------------------------------- |
| **展示型** | 纯 UI、无状态、数据来自 props | UserCard、StatusBadge           |
| **容器型** | 含状态/逻辑、组合多个组件     | UserList、Dashboard             |
| **功能型** | 复用逻辑、无 UI               | useTable、useForm（Composable） |

---

## 大组件拆分

### 拆分条件

- **主要**：业务逻辑复杂（3+ 个独立业务逻辑、混合多种状态管理）
- **辅助**：代码行数 > 1300 行

### 拆分策略

```
按 UI 区域拆分（第一层）
  ↓
按关注点分离（第二层）
```

### 目录结构示例

```
UserTable/
├── UserTable.vue            # 主组件
├── UserTableHeader.vue      # 表头
├── UserTableBody.vue        # 表体
├── UserTableFilters.vue     # 筛选
└── composables/
    ├── useTableData.ts      # 数据逻辑
    └── useTableFilters.ts   # 筛选逻辑
```

### 拆分原则

1. **UI 优先**：先按视觉区域划分，再考虑逻辑拆分
2. **数据下沉**：数据获取逻辑放在最底层的组件或 Composable
3. **事件上浮**：用户交互事件向上传递到业务处理层
4. **接口清晰**：子组件通过 props/emits 与父组件通信

---

## Props 与 Emits 最佳实践

### Props 定义

- 必须使用基于类型的声明 `defineProps<Props>()`
- 必须为可选属性提供默认值 `withDefaults`
- 避免传递过多的 Props（> 10 个），考虑将相关属性组合成一个对象

```typescript
interface UserProps {
   id: number
   name: string
   role?: 'admin' | 'user'
}

const props = withDefaults(defineProps<UserProps>(), {
   role: 'user'
})
```

### Emits 定义

- 必须使用基于类型的声明 `defineEmits<{ (e: 'event', payload: Type): void }>()`
- 事件命名使用 camelCase（如 `updateData`），在模板中监听时使用 kebab-case（如 `@update-data`）

```typescript
const emit = defineEmits<{
   updateData: [data: UserProps]
   delete: [id: number]
}>()
```

---

## 跨层级通信 (Provide / Inject)

当组件嵌套层级较深（> 3 层）时，避免使用 Props 逐层传递（Props Drilling），应使用 `Provide / Inject`。

### 最佳实践

1. **使用 Symbol 作为 Key**：避免命名冲突，并提供类型推导。
2. **保持响应性**：传递 `ref` 或 `reactive` 对象。
3. **单向数据流**：在 Provide 方提供修改数据的方法，Inject 方只读数据并调用方法修改。

```typescript
// context.ts
import type { InjectionKey, Ref } from 'vue'

export interface UserContext {
   user: Ref<User | null>
   updateUser: (newUser: User) => void
}

export const UserContextKey: InjectionKey<UserContext> = Symbol('UserContext')

// Parent.vue
import { provide, ref } from 'vue'
import { UserContextKey } from './context'

const user = ref<User | null>(null)
const updateUser = (newUser: User) => {
   user.value = newUser
}

provide(UserContextKey, { user, updateUser })

// Child.vue
import { inject } from 'vue'
import { UserContextKey } from './context'

const { user, updateUser } = inject(UserContextKey)!
```
