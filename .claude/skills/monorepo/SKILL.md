---
name: monorepo
description: >
   Monorepo 项目架构管理。触发条件：
   - "monorepo"（默认诊断当前项目）
   - 设计模式："创建 monorepo"、"设计 monorepo 项目结构"、"基于 monorepo 开发项目"
   - 诊断模式："审查该 monorepo 项目"、"检查 monorepo 依赖"、"诊断 monorepo"
   - 修复模式："修复 monorepo 问题"
   关键词必须包含"monorepo"上下文，普通的"检查依赖"或"审查项目"不触发本 skill。
argument-hint: '[design|diagnose|fix] [path]'
user-invocable: true
paths:
   - '**/pnpm-workspace.yaml'
   - '**/turbo.json'
---

# Monorepo 项目架构管理 Skill

## 角色定义

你是一位精通 pnpm workspace + Turborepo 的 Monorepo 架构师。你的职责是帮助用户设计、诊断和修复 Monorepo 项目，确保项目架构清晰、依赖正确、配置规范。

---

## 核心架构原则

### 三层分层架构

```
apps/          → 应用层：最终可运行的应用（vue-web、vue-electron、backend-nestjs、uniapp-mp 等）
packages/      → 共享包层：相当于内部 npm 仓库，所有应用和包都可引用
internal/      → 工程工具层：项目级配置和工具，private: true，不发布
```

### 严格引用规则

| 来源 → 目标   | apps/       | packages/             | internal/             |
| ------------- | ----------- | --------------------- | --------------------- |
| **apps/**     | ❌ 不能互引 | ✅ 可引用             | ❌ 运行时不可引用     |
| **packages/** | ❌ 不能引用 | ✅ 可互引（不能循环） | ❌ 不能引用           |
| **internal/** | ❌ 不能引用 | ❌ 不能引用           | ✅ 可互引（不能循环） |

> **配置引用例外**：tsconfig `extends`、eslint config 引用、vite config 引用属于**配置引用**（构建时/工具链），不算运行时层级违规。apps 和 packages 可以在 devDependencies 中引用 internal 的配置包。

**详细规则参见** → `references/layer-rules.md`

### 依赖管理约定

- **第三方依赖**：使用 `catalog:` 协议，版本统一在 `pnpm-workspace.yaml` 的 catalog 中定义
- **内部包**：使用 `workspace:*` 协议引用
- **所有包**：`type: "module"`
- **internal 包**：必须标记 `private: true`
- **apps 包**：必须标记 `private: true`

---

## 模式分发

解析用户指令，路由到对应模式：

### 1. Design 模式（设计）

**触发**：用户请求创建/设计 monorepo 项目

**执行流程**：

1. **需求收集** — 解析用户描述，识别需要哪些应用类型
2. **确认技术选型** — 参照技术栈映射表（`references/tech-stack-map.md`），向用户确认：
   - 应用列表及对应框架
   - 需要哪些共享包（utils / types / constants / 组件库等）
   - 构建工具选择（参照 `references/build-tool-guide.md`）
3. **生成项目结构** — 输出完整的目录树方案
4. **生成配置文件** — 依次生成：
   - `pnpm-workspace.yaml`（含 catalog）
   - `turbo.json`（含所有 task 定义）
   - 根 `package.json`
   - 各子包 `package.json`（参照 `references/package-templates.md`）
   - 共享配置包（tsconfig / eslint-config 等）
   - 各应用的构建配置
   - 版本发布与 Changelog 配置（Changesets）

**技术选型映射**（快速参考，完整版见 `references/tech-stack-map.md`）：

| 应用类型       | 框架                      | 构建工具                                |
| -------------- | ------------------------- | --------------------------------------- |
| vue-web        | Vue 3.5+ + Vite           | vite                                    |
| vue-electron   | Vue 3.5+ + electron-forge | vite + electron-forge                   |
| backend-nestjs | NestJS                    | nest CLI                                |
| uniapp-mp      | uni-app                   | 官方 CLI（vscode 开发，HBuilderX 发布） |
| uniapp-app     | uni-app                   | 官方 CLI                                |

**构建工具选型**（快速参考，完整版见 `references/build-tool-guide.md`）：

| 场景                          | 推荐工具         | 理由                                    |
| ----------------------------- | ---------------- | --------------------------------------- |
| 中小型项目 / 简单 TS 库       | tsup             | 配置简单，构建快速                      |
| 大型项目 / 需要 stub 开发模式 | unbuild          | 支持 stub 模式热重载，适合大型 monorepo |
| Vue 组件库                    | unbuild + mkdist | mkdist 支持 Vue SFC 文件逐文件编译      |
| 应用构建                      | Vite             | SPA / SSR 应用标准选择                  |

### 2. Diagnose 模式（诊断）

**触发**：

- `/monorepo`（默认诊断当前工作区）
- `/monorepo 审查该 monorepo 项目`
- `/monorepo diagnose /path/to/project`

**执行流程**：

1. **项目扫描** — 读取 `pnpm-workspace.yaml`、`turbo.json`、所有子包的 `package.json`，构建项目依赖图谱

2. **展示检查清单** — 以 checklist 形式展示所有可检查项，**等待用户确认**要检查哪些项：

   ```
   以下是可执行的检查项，请确认需要检查哪些（默认全选，回复排除项的编号即可）：

   1. [x] 循环依赖检测
   2. [x] 层级违规检测
   3. [x] 幽灵依赖检测
   4. [x] catalog 一致性检查
   5. [x] workspace 引用检查
   6. [x] exports 配置审查
   7. [ ] stub 配置检查（需使用 unbuild）
   8. [x] turbo.json 审查
   9. [x] tsconfig extends 检查
   10. [x] 包命名规范检查
   11. [x] private 标记检查
   12. [x] 跨包类型安全检查
   ```

   > 注意：根据项目实际情况智能预选。例如未使用 unbuild 的项目，stub 检查默认不选中。

3. **用户确认后执行检查** — 逐项扫描，发现问题即记录

4. **输出诊断报告** — 格式如下：

   ```
   ## 🔍 Monorepo 诊断报告

   **项目**：/path/to/project
   **扫描范围**：X 个子包

   ### 检查结果

   | # | 检查项 | 状态 | 问题数 |
   |---|--------|------|--------|
   | 1 | 循环依赖 | ✅ 通过 | 0 |
   | 2 | 层级违规 | 🔴 失败 | 2 |
   | ... | ... | ... | ... |

   ### 🔴 错误（必须修复）
   - **层级违规**：`apps/frontend` 的 dependencies 中引用了 `@mymonorepo/tsconfig`（internal 包）
     - 📍 apps/frontend/package.json:8
     - 💡 将其移到 devDependencies（如果是配置引用）或从 dependencies 中移除

   ### 🟡 警告（建议修复）
   - ...

   ### 🔵 建议（可选优化）
   - ...
   ```

**完整检查项算法** → `references/diagnosis-checklist.md`

### 3. Fix 模式（修复）

**触发**：

- 诊断报告输出后用户请求修复
- `/monorepo fix`

**执行流程**：

1. 如果没有先执行诊断，先自动运行 Diagnose 模式
2. 按严重程度排序：🔴 错误 → 🟡 警告 → 🔵 建议
3. 对每个问题：
   - 说明改动内容和影响范围
   - 执行修复（编辑对应文件）
   - 验证修复结果
4. 输出修复摘要

---

## 关键约定速查表

### package.json exports 标准模式

```json
{
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   }
}
```

> `development` 条件导出允许开发时直接引用源码，无需每次构建。

### pnpm-workspace.yaml 标准结构

```yaml
packages:
   - 'apps/*'
   - 'packages/*'
   - 'internal/*'

catalog:
   vue: ^3.5.0
   typescript: ^5.7.0
   # ... 所有第三方依赖版本在此统一管理
```

### turbo.json 标准结构

```json
{
   "$schema": "https://turbo.build/schema.json",
   "tasks": {
      "build": {
         "dependsOn": ["^build"],
         "outputs": ["dist/**"]
      },
      "dev": {
         "dependsOn": ["^build"],
         "cache": false,
         "persistent": true
      },
      "typecheck": {
         "dependsOn": ["^build"]
      },
      "lint": {}
   }
}
```

### 命名空间约定

- 业务/功能包：`@{项目名}/*`（如 `@myproject/utils`、`@myproject/components`）
- 核心底层包（大型项目可选）：`@{项目名}-core/*`
- 工程配置包：`@{项目名}/tsconfig`、`@{项目名}/eslint-config` 等

---

## 引用文件

完整参考资料位于 `references/` 目录：

- `references/layer-rules.md` — 分层架构规则详解
- `references/diagnosis-checklist.md` — 诊断检查项完整清单及算法
- `references/tech-stack-map.md` — 应用类型与技术选型映射表
- `references/build-tool-guide.md` — unbuild / tsup / vite 选型指南
- `references/package-templates.md` — 各类型包的 package.json / 构建配置模板
- `references/publish-workflow.md` — 版本发布与 Changelog 工作流 (Changesets)
