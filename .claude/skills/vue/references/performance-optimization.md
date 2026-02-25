# Vue 性能优化指南

## 响应式开销优化

### shallowRef vs ref

当处理大型对象或数组（如从 API 获取的庞大列表），且不需要深层响应性时，使用 `shallowRef` 替代 `ref`。

```typescript
import { shallowRef, ref } from 'vue'

// ❌ 错误：Vue 会递归遍历整个数组，将每个对象的每个属性都转换为响应式，开销巨大
const largeList = ref<LargeItem[]>([])

// ✅ 正确：只有 largeList.value 的重新赋值会触发更新，内部属性修改不会触发
const largeList = shallowRef<LargeItem[]>([])

// 更新数据时：
largeList.value = await fetchLargeData()
```

### v-once 与 v-memo

对于不需要更新的静态内容，使用 `v-once`。
对于依赖特定条件才更新的复杂子树，使用 `v-memo`。

```vue
<!-- ✅ 静态内容，只渲染一次 -->
<div v-once>
  <h1>{{ staticTitle }}</h1>
  <p>{{ staticDescription }}</p>
</div>

<!-- ✅ 仅当 item.id 或 item.status 改变时才重新渲染该 <li> -->
<li v-for="item in list" :key="item.id" v-memo="[item.id, item.status]">
  {{ item.name }} - {{ item.status }}
</li>
```

---

## 组件加载优化

### 异步组件 (Async Components)

对于非首屏必需的组件（如弹窗、抽屉、复杂图表），使用 `defineAsyncComponent` 进行懒加载。

```vue
<script setup lang="ts">
import { defineAsyncComponent, ref } from 'vue'

// ✅ 懒加载组件，只有在渲染时才会下载对应的 JS 文件
const HeavyChart = defineAsyncComponent(() => import('./HeavyChart.vue'))

const showChart = ref(false)
</script>

<template>
   <button @click="showChart = true">显示图表</button>
   <!-- 点击按钮后才会加载 HeavyChart -->
   <HeavyChart v-if="showChart" />
</template>
```

### 路由懒加载

在 Vue Router 中，始终使用动态导入来懒加载路由组件。

```typescript
const routes = [
   {
      path: '/dashboard',
      // ✅ 路由懒加载
      component: () => import('@/views/Dashboard.vue')
   }
]
```

---

## 列表渲染优化

### 虚拟滚动 (Virtual Scrolling)

当渲染超过 100-200 条数据的长列表时，必须使用虚拟滚动（如 `vue-virtual-scroller` 或组件库自带的 Virtual Table/List）。

```vue
<!-- ❌ 错误：直接渲染 10000 条数据会导致浏览器卡顿 -->
<div v-for="item in 10000" :key="item.id">{{ item.name }}</div>

<!-- ✅ 正确：使用虚拟列表组件 -->
<VirtualList :data="largeList" :item-height="50">
  <template #default="{ item }">
    <div>{{ item.name }}</div>
  </template>
</VirtualList>
```

### 稳定的 Key

`v-for` 必须提供唯一且稳定的 `key`（通常是数据的 ID）。绝对不要使用数组索引作为 `key`，除非列表是完全静态的。

```vue
<!-- ❌ 错误：使用 index 作为 key，在列表重排、删除时会导致严重的性能问题和状态错乱 -->
<li v-for="(item, index) in list" :key="index">...</li>

<!-- ✅ 正确：使用唯一 ID -->
<li v-for="item in list" :key="item.id">...</li>
```

---

## 计算属性与侦听器

### 避免在模板中调用复杂函数

模板中的函数在每次组件重新渲染时都会执行。复杂的计算应使用 `computed` 缓存。

```vue
<script setup lang="ts">
import { computed } from 'vue'

const list = ref([...])

// ❌ 错误：每次渲染都会执行 filter 和 sort
const getActiveItems = () => list.value.filter(i => i.active).sort()

// ✅ 正确：只有 list 改变时才会重新计算
const activeItems = computed(() => list.value.filter(i => i.active).sort())
</script>

<template>
   <!-- ❌ 错误 -->
   <div v-for="item in getActiveItems()" :key="item.id">...</div>

   <!-- ✅ 正确 -->
   <div v-for="item in activeItems" :key="item.id">...</div>
</template>
```
