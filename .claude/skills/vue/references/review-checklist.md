# Vue 代码审查清单

使用 `/vue <path>` 审查时，按以下清单逐项检查。

---

## 1. 组件结构

- [ ] 单文件行数 < 1300 行（超过必须拆分）
- [ ] 模板代码 < 500 行
- [ ] 条件嵌套 < 5 层
- [ ] props 数量 < 15 个（超过考虑用配置对象）
- [ ] 业务逻辑复杂时已拆分为子组件

**详细规则** → `component-types.md`

---

## 2. TypeScript 类型

- [ ] Props 使用 `defineProps<Props>()` 定义类型
- [ ] Emits 使用 `defineEmits<Emits>()` 定义类型
- [ ] 无 `any` 类型（必须有明确类型）
- [ ] 异步函数有返回类型
- [ ] 可选 props 使用 `withDefaults()` 设置默认值

```vue
<!-- ✅ 正确 -->
<script setup lang="ts">
interface Props {
   name: string
   count?: number
}

const props = withDefaults(defineProps<Props>(), {
   count: 0
})
</script>

<!-- ❌ 错误 -->
<script setup lang="ts">
const props = defineProps(['name', 'count'])
</script>
```

---

## 3. 组件封装

- [ ] 二次封装时透传 `$attrs`
- [ ] 二次封装时透传所有插槽
- [ ] 继承原组件类型（TypeScript 提示完整）
- [ ] 使用 `defineExpose` 暴露必要方法
- [ ] 无不必要的二次封装（仅样式调整不需要封装）

**详细规则** → `wrapper-pattern.md`

---

## 4. Composable

- [ ] 职责单一（一个 Composable 只做一件事）
- [ ] 调用 API 层，不直接使用 axios
- [ ] 异步操作有 try-catch 错误处理
- [ ] 返回值响应性正确（未解构 reactive）
- [ ] 导出参数类型和返回类型

**详细规则** → `composable-rules.md`

---

## 5. 性能

- [ ] 长列表（>100 项）使用虚拟滚动
- [ ] `v-for` 都有唯一 `key` 属性
- [ ] 重复计算使用 `computed` 缓存
- [ ] 大型组件使用 `defineAsyncComponent` 异步加载
- [ ] 静态内容考虑使用 `v-once`

```vue
<!-- ✅ 正确 -->
<template>
   <div v-for="item in items" :key="item.id">
      {{ item.name }}
   </div>
</template>

<!-- ❌ 错误：使用索引作为 key -->
<template>
   <div v-for="(item, index) in items" :key="index">
      {{ item.name }}
   </div>
</template>
```

---

## 6. 现代语法

- [ ] Vue 3.3+ 使用 `defineModel` 简化 v-model
- [ ] Vue 3.5+ 使用 `useTemplateRef` 替代字符串 ref
- [ ] 使用 `useId()` 生成唯一 ID
- [ ] 需要时使用泛型组件

**详细规则** → `modern-features.md`

---

## 7. 代码规范

- [ ] 复杂逻辑有注释说明
- [ ] 命名语义化（组件名、变量名、函数名）
- [ ] 无重复代码（>10 行相同代码应提取）
- [ ] 组件文件名使用 PascalCase
- [ ] Composable 文件名使用 `use` 前缀

---

## 8. 可访问性 (a11y)

- [ ] 表单元素有关联的 `<label>`
- [ ] 图片有 `alt` 属性
- [ ] 可点击元素可键盘访问（button 或 tabindex）
- [ ] 外部链接使用 `rel="noopener noreferrer"`

```vue
<!-- ✅ 正确 -->
<label :for="inputId">用户名</label>
<input :id="inputId" type="text" />

<img src="logo.png" alt="公司 Logo" />
<a href="https://external.com" rel="noopener noreferrer">外部链接</a>

<!-- ❌ 错误 -->
<div @click="handleClick">点击</div>
<!-- 应该使用 <button> -->
```

---

## 9. 测试

- [ ] 核心组件有单元测试
- [ ] Composable 有单元测试
- [ ] 测试覆盖主要业务逻辑
