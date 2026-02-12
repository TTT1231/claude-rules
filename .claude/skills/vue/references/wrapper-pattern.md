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

## 检查要点

- [ ] 使用 `v-bind="$attrs"` 或 `v-bind="attrs"` 透传属性
- [ ] 使用 `useSlots()` 透传所有插槽
- [ ] 继承原组件类型（`extends ButtonProps`）
- [ ] 使用 `defineExpose` 暴露原组件方法
- [ ] 添加 `/* @vue-ignore */` 避免类型冲突警告

---

## 常见错误

```vue
<!-- ❌ 错误：没有透传插槽 -->
<ElButton v-bind="$attrs" />

<!-- ❌ 错误：解构失去响应性 -->
const { customProp } = toRefs(props)

<!-- ✅ 正确：完整透传 -->
<ElButton ref="buttonRef" v-bind="attrs">
   <slot v-for="(_, name) in slots" :name="name" />
</ElButton>
```
