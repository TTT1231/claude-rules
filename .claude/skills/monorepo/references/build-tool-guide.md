# 构建工具选型指南

本文档帮助在 tsup、unbuild、Vite 之间做出正确的选型决策。

---

## 工具对比总览

| 特性           | tsup                    | unbuild                    | Vite (lib mode)   |
| -------------- | ----------------------- | -------------------------- | ----------------- |
| **定位**       | 简单高效的 TS/JS 库打包 | 统一的 JS 构建系统         | 应用构建 + 库模式 |
| **底层**       | esbuild                 | rollup + esbuild           | rollup + esbuild  |
| **配置复杂度** | ⭐ 极简                 | ⭐⭐ 简单                  | ⭐⭐⭐ 中等       |
| **构建速度**   | ⚡ 极快                 | ⚡ 快                      | ⚡ 快             |
| **stub 模式**  | ❌ 不支持               | ✅ 支持（--stub）          | ❌ 不支持         |
| **Vue SFC**    | ❌ 不支持               | ✅ 支持（mkdist）          | ✅ 支持           |
| **ESM + CJS**  | ✅                      | ✅                         | ✅                |
| **类型声明**   | ✅ (--dts)              | ✅ (自动)                  | 需要插件          |
| **watch 模式** | ✅                      | ✅                         | ✅                |
| **适用场景**   | 中小型 TS 库            | 大型 monorepo / 工程配置包 | SPA 应用 / 库模式 |

---

## 选型决策树

```
需要构建什么？
│
├─ 应用（SPA / SSR）
│  └→ Vite
│
├─ 库包 / 工具包
│  │
│  ├─ 包含 Vue SFC 组件？
│  │  ├─ 是 → unbuild + mkdist
│  │  └─ 否 ↓
│  │
│  ├─ 项目规模？
│  │  ├─ 大型（>10个包）/ 需要 stub 开发模式
│  │  │  └→ unbuild
│  │  └─ 中小型 / 简单 TS 库
│  │     └→ tsup
│  │
│  └─ 工程配置包（internal/）？
│     └→ unbuild（支持 stub，开发体验好）
│
└─ 都不是
   └→ 可能不需要构建（纯类型包等）
```

**总结简化版**：

- **默认选 tsup**：中小型项目，简单 TS/JS 库
- **选 unbuild**：大型项目需要 stub 模式、Vue 组件库（mkdist）、工程配置包
- **选 Vite**：应用构建

---

## tsup 详解

### 适用场景

- 纯 TypeScript / JavaScript 库
- 工具函数包（utils）
- 不需要 stub 模式的中小型项目
- 需要快速打包、零配置体验

### 配置模板

**tsup.config.ts**：

```ts
import { defineConfig } from 'tsup'

export default defineConfig({
   entry: ['src/index.ts'],
   format: ['esm'],
   dts: true,
   clean: true,
   sourcemap: true
})
```

**多入口配置**：

```ts
import { defineConfig } from 'tsup'

export default defineConfig({
   entry: {
      index: 'src/index.ts',
      utils: 'src/utils/index.ts'
   },
   format: ['esm'],
   dts: true,
   clean: true
})
```

**package.json scripts**：

```json
{
   "scripts": {
      "build": "tsup",
      "dev": "tsup --watch"
   }
}
```

### catalog 条目

```yaml
# pnpm-workspace.yaml catalog
catalog:
   tsup: ^8.0.0
```

---

## unbuild 详解

### 适用场景

- 大型 monorepo 中的库包
- 需要 stub 开发模式（修改源码不需要重新构建）
- Vue 组件库（使用 mkdist builder 逐文件编译 SFC）
- internal/ 下的工程配置包

### stub 模式说明

**什么是 stub 模式**：

unbuild 的 `--stub` 选项会生成一个轻量级的代理文件，它不做实际构建，而是将 import 直接重定向到源码文件。这意味着：

1. 修改源码后**无需重新构建**，其他包立即获取最新代码
2. 加快开发迭代速度（大型 monorepo 中尤为明显）
3. 代价是运行时需要 jiti 做实时转译（仅开发时）

**工作流程**：

```bash
# 安装依赖后自动 stub 所有包
pnpm install
# → postinstall 触发 → pnpm -r run stub --if-present
# → 每个使用 unbuild 的包生成 stub 代理文件

# 开发时直接修改源码，无需 build
# 其他包的 import 通过 stub 代理直接访问源码

# 发布/部署前执行真实构建
pnpm build
```

### 配置模板

**基础库 build.config.ts**：

```ts
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig({
   entries: ['src/index'],
   clean: true,
   declaration: true,
   rollup: {
      emitCJS: false
   }
})
```

**Vue 组件库 build.config.ts**（使用 mkdist）：

```ts
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig([
   {
      entries: [
         { builder: 'mkdist', input: './src', pattern: ['**/*.vue'], loaders: ['vue'] },
         { builder: 'mkdist', input: './src', pattern: ['**/*.ts'], format: 'esm', loaders: ['js'] }
      ],
      clean: true,
      declaration: true
   }
])
```

**工程配置包 build.config.ts**：

```ts
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig({
   entries: ['src/index'],
   clean: true,
   declaration: false, // 配置包通常不需要类型声明
   rollup: {
      emitCJS: false
   }
})
```

**package.json scripts**：

```json
{
   "scripts": {
      "build": "unbuild",
      "stub": "unbuild --stub",
      "postinstall": "pnpm run stub"
   }
}
```

### 统一 stub（根级配置）

在根 `package.json` 中配置统一的 postinstall：

```json
{
   "scripts": {
      "postinstall": "pnpm -r run stub --if-present"
   }
}
```

> `--if-present` 确保没有 stub 脚本的包不会报错。

### catalog 条目

```yaml
# pnpm-workspace.yaml catalog
catalog:
   unbuild: ^3.0.0
```

---

## Vite 详解

### 适用场景

- SPA 应用（Vue / React 等）
- 需要 HMR 本地开发服务器
- 库模式构建（可选，但 tsup/unbuild 通常更合适）

### 应用构建配置

**vite.config.ts**（Vue 应用）：

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
   plugins: [vue()],
   resolve: {
      alias: {
         '@': '/src'
      }
   }
})
```

**使用共享 Vite 配置**（大型项目推荐）：

```ts
// apps/vue-web/vite.config.ts
import { defineApplicationConfig } from '@myproject/vite-config'

export default defineApplicationConfig({
   // 应用特有配置
})
```

```ts
// internal/vite-config/src/application.ts
import vue from '@vitejs/plugin-vue'
import type { UserConfig } from 'vite'

export function defineApplicationConfig(options: Partial<UserConfig> = {}): UserConfig {
   return {
      plugins: [vue()],
      resolve: {
         alias: {
            '@': '/src'
         }
      },
      ...options
   }
}
```

---

## 混合使用策略

在同一个 monorepo 中，不同类型的包可以使用不同的构建工具：

```
apps/
  vue-web/               → Vite（应用构建）
  vue-electron/          → Vite + electron-forge
  backend-nestjs/        → nest CLI（tsc）

packages/
  utils/                 → tsup（简单 TS 库）
  types/                 → 无需构建（纯类型导出）
  components/            → unbuild + mkdist（Vue 组件）
  constants/             → tsup（简单 TS 库）

internal/
  tsconfig/              → 无需构建（纯 JSON 导出）
  eslint-config/         → unbuild（stub 模式）
  vite-config/           → unbuild（stub 模式）
```

### catalog 中统一管理构建工具版本

```yaml
# pnpm-workspace.yaml
catalog:
   # 构建工具
   tsup: ^8.0.0
   unbuild: ^3.0.0
   vite: ^6.0.0

   # Vite 插件
   '@vitejs/plugin-vue': ^5.0.0

   # TypeScript
   typescript: ^5.7.0
   vue-tsc: ^2.0.0
```
