# 应用类型与技术选型映射表

本文档定义了 Design 模式中，各应用类型对应的推荐技术栈、目录结构和关键配置。

---

## 应用类型映射

### 前端应用

| 应用类型     | 框架                      | 构建工具              | tsconfig 继承        | 备注                                  |
| ------------ | ------------------------- | --------------------- | -------------------- | ------------------------------------- |
| vue-web      | Vue 3.5+ + Vite           | vite                  | web.json             | 标准 SPA，支持 Vue Router + Pinia     |
| vue-electron | Vue 3.5+ + electron-forge | vite + electron-forge | web.json + node.json | 渲染进程用 Vite，主进程需要 node.json |
| uniapp-mp    | uni-app                   | 官方 CLI              | web.json             | 小程序端，vscode 开发，HBuilderX 发布 |
| uniapp-app   | uni-app                   | 官方 CLI              | web.json             | APP 端，同上                          |

### 后端应用

| 应用类型       | 框架   | 构建工具       | tsconfig 继承 | 备注                                           |
| -------------- | ------ | -------------- | ------------- | ---------------------------------------------- |
| backend-nestjs | NestJS | nest CLI / tsc | node.json     | Node.js 后端，可引用 packages 中的纯 TS 工具包 |

---

## 各应用类型的详细配置

### vue-web

**目录结构**：

```
apps/vue-web/
  index.html
  package.json
  tsconfig.json
  vite.config.ts
  public/
  src/
    App.vue
    main.ts
    assets/
    components/
    composables/
    layouts/
    pages/           # 或 views/
    router/
    stores/
    styles/
```

**package.json 要点**：

```json
{
   "name": "@myproject/vue-web",
   "private": true,
   "type": "module",
   "scripts": {
      "dev": "vite",
      "build": "vue-tsc --noEmit && vite build",
      "preview": "vite preview"
   },
   "dependencies": {
      "vue": "catalog:",
      "vue-router": "catalog:",
      "pinia": "catalog:",
      "@myproject/utils": "workspace:*",
      "@myproject/components": "workspace:*"
   },
   "devDependencies": {
      "vite": "catalog:",
      "@vitejs/plugin-vue": "catalog:",
      "@myproject/tsconfig": "workspace:*",
      "@myproject/vite-config": "workspace:*"
   }
}
```

**tsconfig.json**：

```json
{
   "extends": "@myproject/tsconfig/web.json",
   "compilerOptions": {
      "baseUrl": ".",
      "paths": {
         "@/*": ["src/*"]
      }
   },
   "include": ["src/**/*.ts", "src/**/*.vue", "env.d.ts"]
}
```

---

### vue-electron

**目录结构**：

```
apps/vue-electron/
  package.json
  tsconfig.json
  forge.config.ts        # electron-forge 配置
  vite.main.config.ts    # 主进程 vite 配置
  vite.renderer.config.ts # 渲染进程 vite 配置
  vite.preload.config.ts  # preload 脚本 vite 配置
  src/
    main/                 # 主进程代码
      index.ts
    preload/              # preload 脚本
      index.ts
    renderer/             # 渲染进程（Vue 应用）
      App.vue
      main.ts
      index.html
      components/
      pages/
      router/
      stores/
```

**package.json 要点**：

```json
{
   "name": "@myproject/vue-electron",
   "private": true,
   "type": "module",
   "main": ".vite/build/main.js",
   "scripts": {
      "dev": "electron-forge start",
      "build": "electron-forge make",
      "package": "electron-forge package"
   },
   "dependencies": {
      "vue": "catalog:",
      "vue-router": "catalog:",
      "@myproject/utils": "workspace:*"
   },
   "devDependencies": {
      "electron": "catalog:",
      "@electron-forge/cli": "catalog:",
      "@electron-forge/plugin-vite": "catalog:",
      "vite": "catalog:",
      "@vitejs/plugin-vue": "catalog:",
      "@myproject/tsconfig": "workspace:*"
   }
}
```

---

### backend-nestjs

**目录结构**：

```
apps/backend-nestjs/
  package.json
  tsconfig.json
  tsconfig.build.json
  nest-cli.json
  src/
    main.ts
    app.module.ts
    app.controller.ts
    app.service.ts
    modules/
      users/
        users.module.ts
        users.controller.ts
        users.service.ts
        dto/
        entities/
    common/
      filters/
      guards/
      interceptors/
      pipes/
```

**package.json 要点**：

```json
{
   "name": "@myproject/backend-nestjs",
   "private": true,
   "type": "module",
   "scripts": {
      "dev": "nest start --watch",
      "build": "nest build",
      "start:prod": "node dist/main"
   },
   "dependencies": {
      "@nestjs/core": "catalog:",
      "@nestjs/common": "catalog:",
      "@nestjs/platform-express": "catalog:",
      "@myproject/utils": "workspace:*",
      "@myproject/types": "workspace:*",
      "@myproject/constants": "workspace:*"
   },
   "devDependencies": {
      "@nestjs/cli": "catalog:",
      "typescript": "catalog:",
      "@myproject/tsconfig": "workspace:*",
      "@myproject/eslint-config": "workspace:*"
   }
}
```

> ⚠️ 后端应用只能引用纯 TS/JS 的 packages 包（utils、types、constants 等），**不能**引用包含 Vue 组件的前端包。

**tsconfig.json**：

```json
{
   "extends": "@myproject/tsconfig/node.json",
   "compilerOptions": {
      "baseUrl": ".",
      "paths": {
         "@/*": ["src/*"]
      },
      "emitDecoratorMetadata": true,
      "experimentalDecorators": true
   },
   "include": ["src/**/*.ts"]
}
```

---

### uniapp-mp / uniapp-app

**目录结构**：

```
apps/uniapp-mp/
  package.json
  tsconfig.json
  vite.config.ts
  src/
    App.vue
    main.ts
    pages.json
    manifest.json
    uni.scss
    pages/
      index/
        index.vue
    components/
    composables/
    stores/
    static/
```

**package.json 要点**：

```json
{
   "name": "@myproject/uniapp-mp",
   "private": true,
   "type": "module",
   "scripts": {
      "dev:mp-weixin": "uni -p mp-weixin",
      "build:mp-weixin": "uni build -p mp-weixin",
      "dev:mp-alipay": "uni -p mp-alipay",
      "build:mp-alipay": "uni build -p mp-alipay"
   },
   "dependencies": {
      "vue": "catalog:",
      "@dcloudio/uni-app": "catalog:",
      "@dcloudio/uni-components": "catalog:",
      "@myproject/utils": "workspace:*",
      "@myproject/constants": "workspace:*"
   },
   "devDependencies": {
      "vite": "catalog:",
      "@dcloudio/uni-cli-vite": "catalog:",
      "@myproject/tsconfig": "workspace:*"
   }
}
```

> ⚠️ uniapp 项目在 vscode 中开发，但最终发布需要在 HBuilderX 中进行。打包命令由 uni-app 官方 CLI 提供。

---

## 共享包类型映射

### packages/ 通用包

| 包类型     | 用途                     | 可被谁引用                  | 构建工具           |
| ---------- | ------------------------ | --------------------------- | ------------------ |
| utils      | 工具函数                 | 所有 apps 和 packages       | tsup / unbuild     |
| types      | 共享 TypeScript 类型     | 所有 apps 和 packages       | 无需构建（纯类型） |
| constants  | 共享常量                 | 所有 apps 和 packages       | tsup / unbuild     |
| components | Vue 组件库               | 仅前端 apps                 | unbuild + mkdist   |
| hooks      | Vue Composables          | 仅前端 apps 和前端 packages | tsup / unbuild     |
| request    | HTTP 请求封装            | 前端和后端 apps             | tsup / unbuild     |
| stores     | 状态管理（Pinia stores） | 仅前端 apps                 | tsup / unbuild     |

### internal/ 工程配置包

| 包名             | 用途                 | 引用方式                      |
| ---------------- | -------------------- | ----------------------------- |
| tsconfig         | 共享 TypeScript 配置 | tsconfig.json `extends`       |
| eslint-config    | 共享 ESLint 配置     | eslint.config.mjs `import`    |
| prettier-config  | 共享 Prettier 配置   | prettier.config.mjs `import`  |
| vite-config      | 共享 Vite 配置       | vite.config.ts `import`       |
| stylelint-config | 共享 Stylelint 配置  | stylelint.config.mjs `import` |

---

## 根级目录结构模板

```
{project-name}/
  package.json              # 根 package.json（private: true）
  pnpm-workspace.yaml       # workspace 和 catalog 定义
  turbo.json                # Turborepo 任务配置
  .gitignore
  .npmrc                    # pnpm 配置
  README.md
  apps/
    {app-1}/
    {app-2}/
  packages/
    utils/
    types/
    constants/
    {其他共享包}/
  internal/
    tsconfig/
    eslint-config/
    {其他工程配置}/
```

### 根 package.json 模板

```json
{
   "name": "{project-name}",
   "private": true,
   "type": "module",
   "engines": {
      "node": ">=20.0.0",
      "pnpm": ">=9.0.0"
   },
   "packageManager": "pnpm@9.15.0",
   "scripts": {
      "build": "turbo build",
      "dev": "turbo dev",
      "lint": "turbo lint",
      "typecheck": "turbo typecheck",
      "clean": "turbo clean"
   },
   "devDependencies": {
      "turbo": "catalog:"
   }
}
```
