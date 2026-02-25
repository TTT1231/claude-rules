# 二次封装模式

## 何时需要二次封装

**需要封装**：

- 统一样式规范（如统一按钮尺寸、颜色）
- 添加业务逻辑（如防抖、权限控制）
- 组合多个组件库组件

**不需要封装**：

- 仅调整样式（使用 CSS 变量或全局样式解决）
- 简单的 props 传参

---

## 核心实现

### 完整模板

```vue
<script setup lang="ts">
import { ref, useAttrs, useSlots } from 'vue'
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

---

## 处理 v-model

当封装带有 `v-model` 的组件（如 Input、Select）时，需要正确透传 `modelValue` 和 `update:modelValue` 事件。

### Vue 3.4+ 推荐写法 (使用 defineModel)

```vue
<script setup lang="ts">
import { useAttrs, useSlots } from 'vue'
import { ElInput } from 'element-plus'
import type { InputProps } from 'element-plus'

interface Props extends /* @vue-ignore */ InputProps {
   customProp?: string
}

defineOptions({ inheritAttrs: false })

const props = defineProps<Props>()
const attrs = useAttrs()
const slots = useSlots()

// ✅ 使用 defineModel 自动处理 v-model
const modelValue = defineModel<string | number>()
</script>

<template>
   <ElInput v-model="modelValue" v-bind="attrs">
      <template v-for="(_, name) in slots" #[name]="slotProps">
         <slot :name="name" v-bind="slotProps || {}" />
      </template>
   </ElInput>
</template>
```

---

## 动态插槽透传 (Dynamic Slot Forwarding)

在封装复杂组件（如 Table、Tree）时，需要将所有插槽及其作用域参数（Scoped Slots）完整透传给原组件。

```vue
<template>
   <ElTable v-bind="attrs">
      <!-- ✅ 动态透传所有插槽，并保留作用域参数 -->
      <template v-for="(_, name) in slots" #[name]="slotProps">
         <slot :name="name" v-bind="slotProps || {}" />
      </template>
   </ElTable>
</template>
```

---

## 检查要点

- [ ] 使用 `v-bind="$attrs"` 或 `v-bind="attrs"` 透传属性
- [ ] 使用 `useSlots()` 透传所有插槽，并正确处理作用域插槽
- [ ] 继承原组件类型（`extends ButtonProps`）
- [ ] 使用 `defineExpose` 暴露原组件方法
- [ ] 添加 `/* @vue-ignore */` 避免类型冲突警告
- [ ] 正确处理 `v-model`（优先使用 `defineModel`）

---

## 常见错误

```vue
<!-- ❌ 错误：没有透传插槽 -->
<ElButton v-bind="$attrs" />

<!-- ❌ 错误：解构失去响应性 -->
const { customProp } = toRefs(props)

<!-- ✅ 正确：完整透传 -->
<ElButton ref="buttonRef" v-bind="attrs">
   <template v-for="(_, name) in slots" #[name]="slotProps">
      <slot :name="name" v-bind="slotProps || {}" />
   </template>
</ElButton>
```
