---
name: vue
description: 'Vue 组件开发与代码审查技能 - 开发模式：描述需求时生成高质量 Vue 组件/Composable - 审查模式：/vue <path> 审查该路径下所有 Vue 文件'
argument-hint: 描述需求 或 /vue <path>
user-invocable: true
paths:
   - '**/*.vue'
   - '**/composables/**/*.ts'
---

# Vue 组件开发技能

## 功能说明

### 开发模式

当用户描述需求时，生成高质量的 Vue 代码：

```
创建一个用户列表组件
封装 Element Plus 的 Table 组件
写一个 useTable composable
这个组件太大了，帮我拆分
```

**触发条件**：用户描述包含"创建"、"生成"、"封装"、"编写"、"拆分"等关键词

### 审查模式

当用户使用 `/vue <path>` 命令时，审查指定路径下的 Vue 文件：

```
/vue src/components
/vue src/views/Dashboard.vue
/vue                          # 审查当前目录
```

**执行流程**：

1. 扫描指定路径下的所有 `.vue` 文件
2. 按审查清单逐项检查
3. 输出检查结果（通过/未通过）及改进建议

---

## 1. 开发指南

### 1.1 组件分类

| 类型       | 特征                          | 示例                            |
| ---------- | ----------------------------- | ------------------------------- |
| **展示型** | 纯 UI、无状态、数据来自 props | UserCard、StatusBadge           |
| **容器型** | 含状态/逻辑、组合多个组件     | UserList、Dashboard             |
| **功能型** | 复用逻辑、无 UI               | useTable、useForm（Composable） |

### 1.2 二次封装

**何时需要**：封装组件库组件，需要保证原组件的 props/emits 类型提示完整透传

**核心要点**：

```vue
<script setup lang="ts">
import { useAttrs, useSlots } from 'vue'
import { ElButton } from 'element-plus'
import type { ButtonProps, ButtonEmits } from 'element-plus'

// 扩展的 props
interface Props extends /* @vue-ignore */ ButtonProps {
   customProp?: string
}

defineOptions({ inheritAttrs: false })

const props = defineProps<Props>()
const attrs = useAttrs()
const slots = useSlots()

// 扩展的 emits
const emit = defineEmits<{
   customEvent: [value: string]
}>()

// 暴露原组件方法
const buttonRef = ref<InstanceType<typeof ElButton>>()
defineExpose({
   focus: () => buttonRef.value?.focus()
})
</script>

<template>
   <ElButton ref="buttonRef" v-bind="attrs">
      <slot v-for="(_, name) in slots" :name="name" />
   </ElButton>
</template>
```

**检查要点**：

- 使用 `v-bind="$attrs"` 或 `v-bind="attrs"` 透传属性
- 使用 `useSlots()` 透传所有插槽
- 继承原组件类型（`extends ButtonProps`）
- 使用 `defineExpose` 暴露原组件方法

### 1.3 大组件拆分

**拆分条件**：

- **主要**：业务逻辑复杂（3+ 个独立业务逻辑、混合多种状态管理）
- **辅助**：代码行数 > 1300 行

**拆分策略**：

```
按 UI 区域拆分（第一层）
  ↓
按关注点分离（第二层）
```

**目录结构**：

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

### 1.4 Composable 规则

**核心规则**：

```typescript
// ✅ 单一职责
export function useTableData() {
   /* 只处理数据 */
}
export function useTableFilters() {
   /* 只处理筛选 */
}

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

// ✅ 正确返回响应式数据
return { data, loading } // ref 直接返回
return state // reactive 整体返回

// ❌ 错误：解构 reactive 失去响应性
return { ...state }
```

**类型导出**：

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

### 1.5 现代 Vue 特性

**defineModel（Vue 3.3+）**：

```vue
<script setup lang="ts">
// ✅ 简化 v-model
const modelValue = defineModel<string>({ required: true })
const title = defineModel<string>('title', { default: '' })
</script>

<template>
   <input v-model="modelValue" />
</template>
```

**useTemplateRef（Vue 3.5+）**：

```vue
<script setup lang="ts">
const inputRef = useTemplateRef<HTMLInputElement>('input')
</script>

<template>
   <input ref="input" />
</template>
```

**泛型组件**：

```vue
<script setup lang="ts" generic="T">
interface Props {
   data: T[]
   columns: { key: keyof T; label: string }[]
}
defineProps<Props>()
</script>
```

---

## 2. 代码审查清单

使用 `/vue <path>` 审查时，按以下清单逐项检查：

### 2.1 组件结构

- [ ] 单文件行数 < 1300 行（超过必须拆分）
- [ ] 模板代码 < 500 行
- [ ] 条件嵌套 < 5 层
- [ ] props 数量 < 15 个（超过考虑用配置对象）
- [ ] 业务逻辑复杂时已拆分为子组件

### 2.2 TypeScript 类型

- [ ] Props 使用 `defineProps<Props>()` 定义类型
- [ ] Emits 使用 `defineEmits<Emits>()` 定义类型
- [ ] 无 `any` 类型（必须有明确类型）
- [ ] 异步函数有返回类型
- [ ] 可选 props 使用 `withDefaults()` 设置默认值

### 2.3 组件封装

- [ ] 二次封装时透传 `$attrs`
- [ ] 二次封装时透传所有插槽
- [ ] 继承原组件类型（TypeScript 提示完整）
- [ ] 使用 `defineExpose` 暴露必要方法
- [ ] 无不必要的二次封装（仅样式调整不需要封装）

### 2.4 Composable

- [ ] 职责单一（一个 Composable 只做一件事）
- [ ] 调用 API 层，不直接使用 axios
- [ ] 异步操作有 try-catch 错误处理
- [ ] 返回值响应性正确（未解构 reactive）
- [ ] 导出参数类型和返回类型

### 2.5 性能

- [ ] 长列表（>100 项）使用虚拟滚动
- [ ] `v-for` 都有唯一 `key` 属性
- [ ] 重复计算使用 `computed` 缓存
- [ ] 大型组件使用 `defineAsyncComponent` 异步加载
- [ ] 静态内容考虑使用 `v-once`

### 2.6 现代语法

- [ ] Vue 3.3+ 使用 `defineModel` 简化 v-model
- [ ] Vue 3.5+ 使用 `useTemplateRef` 替代字符串 ref
- [ ] 使用 `useId()` 生成唯一 ID
- [ ] 需要时使用泛型组件

### 2.7 代码规范

- [ ] 复杂逻辑有注释说明
- [ ] 命名语义化（组件名、变量名、函数名）
- [ ] 无重复代码（>10 行相同代码应提取）
- [ ] 组件文件名使用 PascalCase
- [ ] Composable 文件名使用 `use` 前缀

### 2.8 可访问性 (a11y)

- [ ] 表单元素有关联的 `<label>`
- [ ] 图片有 `alt` 属性
- [ ] 可点击元素可键盘访问（button 或 tabindex）
- [ ] 外部链接使用 `rel="noopener noreferrer"`

### 2.9 测试

- [ ] 核心组件有单元测试
- [ ] Composable 有单元测试
- [ ] 测试覆盖主要业务逻辑

---

## 3. 快速决策树

```
用户请求 → 判断类型

是纯逻辑复用吗？
  → YES: Composable (useXxx)
  → NO: 继续

是封装组件库组件吗？
  → YES: 二次封装（保证类型透传）
  → NO: 继续

业务逻辑复杂 或 >1300 行？
  → YES: 拆分为子组件 + Composable
  → NO: 单文件组件
```

---

## 4. 审查输出格式

审查完成后，按以下格式输出结果：

```markdown
## 📋 [文件名] 审查结果

### ✅ 通过项

- [x] 组件结构合理
- [x] TypeScript 类型完整
- [x] ...

### ❌ 问题项

- [ ] **[问题标题]**
   - 位置：[文件:行号]
   - 问题：[描述]
   - 建议：[改进方案]

### 📊 总评：X/10
```
