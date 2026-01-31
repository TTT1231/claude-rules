---
paths:
   - '**/*.vue' # Vue 组件
---

# 内存管理

## 适用范围

**Vue 组件**：包括 `<template>`、`<script lang="ts">`、`<script lang="js">`

---

## 数据结构选择

| 场景          | 选择       |
| ------------- | ---------- |
| 对象作为 key  | WeakMap    |
| 需要遍历/size | Map        |
| 临时关联数据  | WeakMap    |
| 持久缓存      | Map + 清理 |

---

### Map vs WeakMap

```ts
// ✅ WeakMap - 自动 GC
const cache = new WeakMap<object, Data>()

// ❌ Map - 必须手动清理
const cache = new Map<object, Data>()
```

---

## 定时器清理

```ts
// ✅ 正确
const timer = setInterval(() => {}, 1000)
onUnmounted(() => clearInterval(timer))

// ❌ 错误 - 内存泄漏
setInterval(() => {}, 1000)
```

---

## 事件监听器清理

```ts
// ✅ 正确
window.addEventListener('resize', handler)
onUnmounted(() => window.removeEventListener('resize', handler))

// ❌ 错误
window.addEventListener('resize', handler)
```

## 第三方实例销毁

```ts
// ✅ 正确
const chart = echarts.init(el)
onUnmounted(() => chart.dispose())

// ❌ 错误
echarts.init(el)
```

---

## 订阅/观察者清理

```ts
// ✅ 正确
const subscription = observable.subscribe(handler)
onUnmounted(() => subscription.unsubscribe())

// ❌ 错误
observable.subscribe(handler)
```

---

## 避免闭包持有组件实例

```ts
// ❌ 错误 - 闭包持有 this
const handler = () => this.doSomething()

// ✅ 正确 - 使用 onUnmounted 清理，统一模式

onUnmounted(() => {
   // 1. 清除定时器
   clearInterval(timer)
   clearTimeout(timer)

   // 2. 移除监听
   window.removeEventListener('resize', handler)

   // 3. 清空 Map
   map.clear()

   // 4. 销毁实例
   instance.dispose()
   instance.destroy()

   // 5. 取消订阅
   subscription.unsubscribe()
})
```

---

## keep-alive 限制缓存数量

```vue
<template>
   // ❌ 错误 - 无限制缓存
   <keep-alive>
      <component :is="currentView" />
   </keep-alive>

   // ✅ 正确 - 限制最大缓存数
   <keep-alive :max="8">
      <component :is="currentView" />
   </keep-alive>
</template>
```

---

## 禁止事项

- ❌ 创建定时器/监听器不清理
- ❌ Map 持有对象引用不置空
- ❌ 第三方库实例不销毁
- ❌ 全局事件总线绑定不解绑
- ❌ 闭包持有组件实例未清理
- ❌ keep-alive 无限制缓存组件
- ❌ 移动端 keep-alive 超过 5 个

---
