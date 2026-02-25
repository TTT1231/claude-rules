---
name: vue
description: 'Vue 组件开发与代码审查技能 - 开发模式：描述需求时生成高质量 Vue 组件/Composable - 审查模式：/vue <path> 审查该路径下所有 Vue 文件'
argument-hint: '描述需求 或 /vue <path>'
user-invocable: true
paths:
   - '**/*.vue'
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

---

## 模式分发

### 1. 开发模式

**执行流程**：

1. **需求解析** — 理解用户想要实现的功能
2. **类型判断** — 使用决策树确定组件类型
3. **代码生成** — 按照规范生成代码
4. **类型提示** — 确保完整的 TypeScript 类型定义

### 2. 审查模式

**执行流程**：

1. **文件扫描** — 使用 Glob 查找指定路径下的所有 `.vue` 和 composables 文件
2. **展示检查清单** — 展示可检查项，等待用户确认：

   ```
   以下是可执行的检查项，请确认需要检查哪些（默认全选，回复排除项的编号即可）：

   1. [x] 组件结构
   2. [x] TypeScript 类型
   3. [x] 组件封装
   4. [x] Composable
   5. [x] 性能
   6. [x] 现代语法
   7. [x] 代码规范
   8. [x] 可访问性 (a11y)
   9. [ ] 测试（默认不选，需明确指定）
   ```

3. **逐项检查** — 按照清单扫描代码，记录问题
4. **输出诊断报告** — 按统一格式输出结果（见下方）

---

## 快速决策树

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

## 代码生成规范 (AI 必须遵守)

1. **语法标准**：
   - 必须使用 `<script setup lang="ts">`
   - 优先使用 `const` 声明变量
   - 必须使用 `async/await` 处理异步操作，并包裹在 `try/catch` 中
   - 优先使用 Vue 3.5+ 新特性（如 `useTemplateRef`, `useId`, `defineModel`）
2. **类型定义**：
   - 必须为所有 Props、Emits、Refs 提供明确的 TypeScript 类型
   - 避免使用 `any`，优先使用 `unknown` 或具体类型
   - 复杂类型定义应提取到单独的 `types.ts` 文件中
3. **状态管理**：
   - 局部状态使用 `ref` 或 `reactive`
   - 跨组件状态优先使用 `Provide/Inject`
   - 全局状态使用 Pinia
4. **性能优化**：
   - 列表渲染必须提供唯一的 `:key`
   - 频繁更新的非响应式数据不要使用 `ref`
   - 庞大的只读数据使用 `shallowRef`

---

## 审查输出格式

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

---

## 引用文件

详细规则位于 `references/` 目录：

- `references/component-types.md` — 组件分类、大组件拆分规则、Props/Emits、Provide/Inject
- `references/wrapper-pattern.md` — 二次封装完整指南、v-model 处理、动态插槽
- `references/composable-rules.md` — Composable 编写规则、副作用清理、状态共享
- `references/modern-features.md` — Vue 3.3+ / 3.5+ 新特性
- `references/review-checklist.md` — 详细的审查清单
- `references/project-setup.md` — Vue Web 项目配置（Pinia, Router, Axios, unplugin 等）
- `references/performance-optimization.md` — 性能优化指南（shallowRef, v-memo, 虚拟滚动等）
