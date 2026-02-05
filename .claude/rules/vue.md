---
paths:
   - '**/*.vue'
---

# Vue Standards

## File Template

```vue
<script setup lang="ts"></script>

<template></template>

<style lang="scss" scoped></style>
```

## Prohibitions

| Practice                              | Explanation                       |
| ------------------------------------- | --------------------------------- |
| `<script setup>` without `lang="ts"`  | Required for TS projects          |
| Missing `<style lang="scss" scoped>`  | Required for non-Tailwind projects |
| Composables not named `useXxx`        | Must use `use` prefix             |
| Props/Emits without type definitions  | TS projects must be type-safe     |
| Static inline styles                  | `<div style="color: red">`        |
