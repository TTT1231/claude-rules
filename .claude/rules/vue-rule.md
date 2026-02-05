---
paths:
   - '**/*.vue'
---

# Vue 规范

## 文件模板

```vue
<script setup lang="ts"></script>

<template></template>

<style lang="scss" scoped></style>
```

## 禁止事项

| 行为                              | 说明                       |
| --------------------------------- | -------------------------- |
| `<script setup>` 不加 `lang="ts"` | TS 项目必须                |
| 缺少 `<style lang="scss" scoped>` | 非 Tailwind 项目必须       |
| Composables 命名非 `useXxx`       | 必须前缀 `use`             |
| Props/Emits 无类型定义            | TS 项目必须类型安全        |
| 静态内联样式                      | `<div style="color: red">` |
