---
paths:
   - 'src/components/**/*.vue' # 单体架构路径
   - 'packages/ui/**/*.vue' # monorepo 路径
---

# 二次组件封装

## 封装模板

```vue
<script setup lang="ts">
import { Button } from 'ant-design-vue'
import { getCurrentInstance, h, type ComponentPublicInstance } from 'vue'
import type { ButtonProps } from 'ant-design-vue'

type ButtonInstance = ComponentPublicInstance<ButtonProps>
const vm = getCurrentInstance()

function changeRef(expose: Element | ButtonInstance | null) {
   if (vm) {
      vm.exposed = expose
   }
}

defineExpose({} as ButtonInstance)
</script>

<template>
   <component :is="h(Button, { ...$attrs, ref: changeRef }, $slots)" />
</template>
```

---

## 五要素

| 要素 | 实现方式                          | 说明                            |
| ---- | --------------------------------- | ------------------------------- |
| 属性 | `{ ...$attrs }`                   | 自动透传所有 props              |
| 事件 | 自动透传                          | 无需处理                        |
| 方法 | `changeRef + defineExpose`        | 暴露原组件实例供父组件 ref 调用 |
| 插槽 | `{ ...$slots }` 或 `$slots`       | 自动透传所有插槽                |
| 类型 | `ComponentPublicInstance<TProps>` | 保持完整 TS 类型提示            |

---

## 禁止事项

- ❌ 使用 div 等容器包裹 <component>（会导致事件冒泡重复触发）
- ❌ 手动 declare props（破坏透传机制）
- ❌ 手动 defineEmits（破坏事件透传）
- ❌ 省略类型定义（失去 TS 智能提示）

## 使用示例

```vue
<!-- 父组件 -->
<script setup lang="ts">
import { ref } from 'vue'
import { MyButton } from './components'

const buttonRef = ref()

const handleClick = () => {
   buttonRef.value?.focus() // 调用原组件方法
}
</script>

<template>
   <MyButton ref="buttonRef" type="primary" @click="handleClick"> 点击 </MyButton>
</template>
```

---
