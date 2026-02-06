# 分层架构规则详解

## 三层架构定义

### apps/ — 应用层

最终可运行、可部署的应用。每个子目录是一个独立应用。

**特征**：

- `private: true`（不发布到 npm）
- 拥有 `dev`、`build`、`preview` 等脚本
- 是依赖图的**叶子节点**（只消费，不被其他包依赖）

**典型内容**：

```
apps/
  vue-web/          # Vue SPA 应用
  vue-electron/     # Electron 桌面应用
  backend-nestjs/   # NestJS 后端服务
  uniapp-mp/        # uni-app 小程序
```

### packages/ — 共享包层

相当于项目的**内部 npm 仓库**。任何应用或其他 packages 下的包都可以引用。

**特征**：

- 可发布（可选，`private: true` 则不发布）
- 拥有完整的 `exports` 字段
- 必须有 `build` 脚本
- 是依赖图的**中间节点**

**典型内容**：

```
packages/
  utils/           # 工具函数（前后端通用）
  types/           # 共享 TypeScript 类型
  constants/       # 共享常量
  components/      # Vue 组件库（仅前端应用可用）
  hooks/           # Vue Composables（仅前端应用可用）
  request/         # HTTP 请求封装
```

### internal/ — 工程工具层

项目级的开发工具和配置，**不发布、不在运行时引用**。

**特征**：

- **必须** `private: true`
- 仅通过**配置引用**方式被使用（非运行时 import）
- 通常使用 unbuild stub 模式开发

**典型内容**：

```
internal/
  tsconfig/         # 共享 TypeScript 配置
  eslint-config/    # 共享 ESLint 配置
  prettier-config/  # 共享 Prettier 配置
  vite-config/      # 共享 Vite 配置
  stylelint-config/ # 共享 Stylelint 配置
```

---

## 引用规则详解

### 规则矩阵

```
apps/        →  packages/*     ✅ 允许（运行时 import）
apps/        →  internal/*     ❌ 禁止运行时 import
apps/        →  apps/*         ❌ 禁止（应用间不能互引）

packages/*   →  packages/*     ✅ 允许（不能循环依赖）
packages/*   →  apps/*         ❌ 禁止
packages/*   →  internal/*     ❌ 禁止运行时 import

internal/*   →  internal/*     ✅ 允许（不能循环依赖）
internal/*   →  apps/*         ❌ 禁止
internal/*   →  packages/*     ❌ 禁止
```

### 配置引用 vs 运行时引用

这是一个关键区分：

#### 配置引用（✅ 允许跨层）

配置引用发生在**构建时/工具链层面**，不会出现在最终产物中：

```jsonc
// tsconfig.json — 配置引用，允许任何包引用 internal/tsconfig
{
   "extends": "@myproject/tsconfig/web.json"
}
```

```js
// eslint.config.mjs — 配置引用
import { defineConfig } from '@myproject/eslint-config'
```

```ts
// vite.config.ts — 配置引用
import { defineApplicationConfig } from '@myproject/vite-config'
```

**判断标准**：出现在以下位置的引用属于配置引用：

- `tsconfig.json` 的 `extends` 字段
- `eslint.config.*` 文件中的 import
- `vite.config.*` / `build.config.*` 中的 import
- `prettier.config.*` / `stylelint.config.*` 中的 import
- `package.json` 的 `devDependencies`（仅开发时依赖）

#### 运行时引用（❌ 必须遵守层级规则）

运行时引用出现在**源码**中（`src/` 目录下的文件），会包含在最终产物中：

```ts
// src/main.ts — 运行时引用，必须遵守层级规则
import { formatDate } from '@myproject/utils' // ✅ packages → packages
import { setupRouter } from '@myproject/router' // ✅ apps → packages
import { viteConfig } from '@myproject/vite-config' // ❌ apps → internal（违规！）
```

**判断标准**：出现在 `src/` 目录下的 `.ts`、`.vue`、`.tsx` 文件中的 import 属于运行时引用。

### 语义层面的跨包安全

即使层级规则允许，还需要考虑**语义合理性**：

```ts
// apps/backend-nestjs/src/main.ts
import { Button } from '@myproject/components' // ❌ 语义违规：后端不应引入前端组件

import { formatDate } from '@myproject/utils' // ✅ 语义合理：工具函数前后端通用
import { API_CODES } from '@myproject/constants' // ✅ 语义合理：常量前后端通用
```

**诊断时的语义检查规则**：

- 后端应用（NestJS 等）不应引用包含 Vue/React 组件的包
- 前端组件包（含 `.vue` 文件的包）不应被后端应用引用
- 工具类包（纯 TS/JS）可被任何应用引用

---

## 诊断时的检查方法

### 检查 1：层级违规

扫描每个子包的 `package.json`：

1. 读取 `dependencies`（运行时依赖）
2. 识别其中的 `workspace:*` 引用
3. 根据当前包所在层级和目标包所在层级，判断是否违反引用矩阵
4. 如果在 `devDependencies` 中引用 internal 包，检查是否属于配置引用（合理的）

### 检查 2：运行时源码违规

扫描 `src/` 目录下的源码文件：

1. 提取所有 import 语句
2. 筛选出引用 workspace 内部包（以项目命名空间开头）的 import
3. 根据层级规则判断是否违规
4. 进行语义检查（前端包 → 后端应用等）

### 检查 3：循环依赖

构建 packages 间的依赖有向图，检测是否存在环。具体算法：

1. 从每个 package.json 的 dependencies/devDependencies 中提取 `workspace:*` 依赖
2. 构建有向图（节点 = 包名，边 = 依赖关系）
3. 使用 DFS 检测环
4. 如果发现环，输出完整的依赖链
