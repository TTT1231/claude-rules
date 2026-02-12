# 现代 Vue 特性

## defineModel（Vue 3.3+）

简化 v-model 双向绑定，无需手动定义 props + emits。

### 基本用法

```vue
<!-- 父组件 -->
<MyComponent v-model="count" />

<!-- 子组件 -->
<script setup lang="ts">
// ✅ 简化写法
const modelValue = defineModel<number>({ required: true })

// ✅ 命名 model
const title = defineModel<string>('title', { default: '' })
</script>

<template>
   <input v-model="modelValue" />
</template>
```

### 对比旧写法

```vue
<!-- ❌ 旧写法：需要 props + emits -->
<script setup lang="ts">
const props = defineProps<{ modelValue: number }>()
const emit = defineEmits<{ 'update:modelValue': [value: number] }>()

const localValue = computed({
   get: () => props.modelValue,
   set: (val) => emit('update:modelValue', val)
})
</script>
```

---

## useTemplateRef（Vue 3.5+）

替代字符串 ref，提供完整的类型推导。

### 基本用法

```vue
<script setup lang="ts">
// ✅ 新写法：类型自动推导
const inputRef = useTemplateRef<HTMLInputElement>('input')

inputRef.value?.focus() // 类型安全
</script>

<template>
   <input ref="input" />
</template>
```

### 对比旧写法

```vue
<!-- ❌ 旧写法：需要泛型参数 -->
<script setup lang="ts">
const inputRef = ref<HTMLInputElement>()

onMounted(() => {
   inputRef.value?.focus()
})
</script>

<template>
   <input ref="inputRef" />
</template>
```

---

## useId()（Vue 3.5+）

生成唯一 ID，用于无障碍标签。

```vue
<script setup lang="ts">
const id = useId()
</script>

<template>
   <label :for="id">用户名</label>
   <input :id="id" type="text" />
</template>
```

---

## 泛型组件（Vue 3.3+）

组件支持泛型参数。

### 基本用法

```vue
<script setup lang="ts" generic="T extends object">
interface Props {
   data: T[]
   columns: { key: keyof T; label: string }[]
}

const props = defineProps<Props>()
</script>

<template>
   <table>
      <tr v-for="item in data" :key="item.id">
         <td v-for="col in columns" :key="String(col.key)">
            {{ item[col.key] }}
         </td>
      </tr>
   </table>
</template>
```

### 使用

```vue
<script setup lang="ts">
interface User {
   id: number
   name: string
   email: string
}

const users: User[] = [...]
const columns = [
   { key: 'name' as const, label: '姓名' },
   { key: 'email' as const, label: '邮箱' }
]
</script>

<template>
   <!-- 类型自动推导 -->
   <DataTable :data="users" :columns="columns" />
</template>
```

---

## defineProps 跨文件类型复用

```typescript
// types.ts
export interface BaseButtonProps {
   loading?: boolean
   disabled?: boolean
}
```

```vue
<script setup lang="ts">
import type { BaseButtonProps } from './types'

interface Props extends BaseButtonProps {
   variant?: 'primary' | 'secondary'
}

defineProps<Props>()
</script>
```
