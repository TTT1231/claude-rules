# 各类型包的 package.json / 构建配置模板

本文档提供 monorepo 中各类型子包的标准 package.json 和构建配置模板。

---

## 模板变量说明

| 变量        | 说明         | 示例         |
| ----------- | ------------ | ------------ |
| `{scope}`   | 项目命名空间 | `@myproject` |
| `{name}`    | 包名         | `utils`      |
| `{project}` | 项目名       | `myproject`  |

---

## 1. 工具库包（packages/utils）

纯 TypeScript 工具函数，前后端通用。

### package.json

```json
{
   "name": "{scope}/utils",
   "version": "1.0.0",
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "sideEffects": false,
   "scripts": {
      "build": "tsup"
   },
   "dependencies": {},
   "devDependencies": {
      "tsup": "catalog:",
      "typescript": "catalog:"
   }
}
```

### tsup.config.ts

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

### 目录结构

```
packages/utils/
  package.json
  tsup.config.ts
  tsconfig.json
  src/
    index.ts           # 统一导出
    date.ts            # 日期工具
    string.ts          # 字符串工具
    type-guards.ts     # 类型守卫
```

---

## 2. 类型包（packages/types）

纯 TypeScript 类型定义，无需构建。

### package.json

```json
{
   "name": "{scope}/types",
   "version": "1.0.0",
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./src/index.ts"
      }
   },
   "main": "./src/index.ts",
   "types": "./src/index.ts",
   "files": ["src"],
   "sideEffects": false,
   "scripts": {},
   "dependencies": {},
   "devDependencies": {
      "typescript": "catalog:"
   }
}
```

> 纯类型包不需要构建步骤，直接导出源码即可。

### 目录结构

```
packages/types/
  package.json
  tsconfig.json
  src/
    index.ts           # 统一导出
    user.ts            # 用户类型
    api.ts             # API 响应类型
    common.ts          # 通用类型
```

---

## 3. 常量包（packages/constants）

共享常量，前后端通用。

### package.json

```json
{
   "name": "{scope}/constants",
   "version": "1.0.0",
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "sideEffects": false,
   "scripts": {
      "build": "tsup"
   },
   "dependencies": {},
   "devDependencies": {
      "tsup": "catalog:",
      "typescript": "catalog:"
   }
}
```

---

## 4. Vue 组件库包（packages/components）

Vue 3 组件库，仅前端应用可引用。

### package.json

```json
{
   "name": "{scope}/components",
   "version": "1.0.0",
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "sideEffects": false,
   "scripts": {
      "build": "unbuild",
      "stub": "unbuild --stub"
   },
   "dependencies": {
      "vue": "catalog:"
   },
   "devDependencies": {
      "unbuild": "catalog:",
      "typescript": "catalog:"
   }
}
```

### build.config.ts

```ts
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig([
   {
      entries: [
         {
            builder: 'mkdist',
            input: './src',
            pattern: ['**/*.vue'],
            loaders: ['vue']
         },
         {
            builder: 'mkdist',
            input: './src',
            pattern: ['**/*.ts'],
            format: 'esm',
            loaders: ['js']
         }
      ],
      clean: true,
      declaration: true
   }
])
```

### 目录结构

```
packages/components/
  package.json
  build.config.ts
  tsconfig.json
  src/
    index.ts               # 统一导出
    button/
      Button.vue
      index.ts
    modal/
      Modal.vue
      index.ts
    table/
      Table.vue
      index.ts
```

---

## 5. Composables 包（packages/hooks）

Vue 3 Composable 函数，仅前端应用可引用。

### package.json

```json
{
   "name": "{scope}/hooks",
   "version": "1.0.0",
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "sideEffects": false,
   "scripts": {
      "build": "tsup"
   },
   "dependencies": {
      "vue": "catalog:"
   },
   "devDependencies": {
      "tsup": "catalog:",
      "typescript": "catalog:"
   }
}
```

---

## 6. 共享 tsconfig 包（internal/tsconfig）

### package.json

```json
{
   "name": "{scope}/tsconfig",
   "version": "1.0.0",
   "private": true,
   "type": "module",
   "exports": {
      "./base.json": "./base.json",
      "./web.json": "./web.json",
      "./node.json": "./node.json",
      "./lib.json": "./lib.json"
   },
   "files": ["*.json"]
}
```

> 纯 JSON 配置包，无需构建工具。

### base.json

```json
{
   "$schema": "https://json.schemastore.org/tsconfig",
   "compilerOptions": {
      "strict": true,
      "target": "ESNext",
      "module": "ESNext",
      "moduleResolution": "Bundler",
      "esModuleInterop": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true,
      "resolveJsonModule": true,
      "isolatedModules": true,
      "verbatimModuleSyntax": true,
      "declaration": true,
      "declarationMap": true,
      "sourceMap": true
   },
   "exclude": ["node_modules", "dist"]
}
```

### web.json

```json
{
   "extends": "./base.json",
   "compilerOptions": {
      "lib": ["ESNext", "DOM", "DOM.Iterable"],
      "jsx": "preserve",
      "jsxImportSource": "vue"
   }
}
```

### node.json

```json
{
   "extends": "./base.json",
   "compilerOptions": {
      "lib": ["ESNext"],
      "types": ["node"]
   }
}
```

### lib.json

```json
{
   "extends": "./base.json",
   "compilerOptions": {
      "lib": ["ESNext"],
      "declaration": true,
      "declarationMap": true
   }
}
```

### 目录结构

```
internal/tsconfig/
  package.json
  base.json
  web.json
  node.json
  lib.json
```

---

## 7. 共享 ESLint 配置包（internal/eslint-config）

### package.json

```json
{
   "name": "{scope}/eslint-config",
   "version": "1.0.0",
   "private": true,
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "scripts": {
      "build": "unbuild",
      "stub": "unbuild --stub"
   },
   "dependencies": {
      "eslint": "catalog:",
      "@eslint/js": "catalog:",
      "eslint-plugin-vue": "catalog:",
      "typescript-eslint": "catalog:"
   },
   "devDependencies": {
      "unbuild": "catalog:",
      "typescript": "catalog:"
   }
}
```

### build.config.ts

```ts
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig({
   entries: ['src/index'],
   clean: true,
   declaration: false,
   rollup: {
      emitCJS: false
   }
})
```

### src/index.ts 示例

```ts
import type { Linter } from 'eslint'

export function defineConfig(options: { vue?: boolean } = {}): Linter.Config[] {
   const configs: Linter.Config[] = [
      // 基础 JS/TS 规则
      // ...
   ]

   if (options.vue !== false) {
      // Vue 规则
      // ...
   }

   return configs
}
```

### 使用方式（各包的 eslint.config.mjs）

```js
import { defineConfig } from '{scope}/eslint-config'

export default defineConfig({ vue: true })
```

---

## 8. 共享 Vite 配置包（internal/vite-config）

### package.json

```json
{
   "name": "{scope}/vite-config",
   "version": "1.0.0",
   "private": true,
   "type": "module",
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "main": "./dist/index.mjs",
   "types": "./src/index.ts",
   "files": ["dist", "src"],
   "scripts": {
      "build": "unbuild",
      "stub": "unbuild --stub"
   },
   "dependencies": {
      "vite": "catalog:",
      "@vitejs/plugin-vue": "catalog:"
   },
   "devDependencies": {
      "unbuild": "catalog:",
      "typescript": "catalog:"
   }
}
```

---

## 9. 应用包模板（apps/\*）

应用包为最终可运行的应用，结构因应用类型不同而异。参见 `tech-stack-map.md` 获取各类型应用的详细模板。

### 通用要求

```json
{
   "name": "{scope}/{app-name}",
   "private": true,
   "type": "module"
}
```

所有 apps 包**必须**：

- 设置 `"private": true`
- 设置 `"type": "module"`
- dependencies 中只引用 `packages/*` 包和第三方包（使用 `catalog:`）
- devDependencies 中引用 `internal/*` 配置包（使用 `workspace:*`）
- 不被其他任何包引用

---

## exports 条件导出说明

### 标准模式

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

| 条件          | 用途                 | 指向                             |
| ------------- | -------------------- | -------------------------------- |
| `types`       | TypeScript 类型解析  | 源码类型（开发）或 .d.ts（发布） |
| `development` | 开发时 import 解析   | 源码（配合 Vite 条件解析）       |
| `import`      | 生产构建 import 解析 | 构建产物                         |

### 多入口导出

```json
{
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      },
      "./utils": {
         "types": "./src/utils/index.ts",
         "development": "./src/utils/index.ts",
         "import": "./dist/utils/index.mjs"
      }
   }
}
```

### 发布时配置（可选）

使用 `publishConfig` 覆盖 exports 指向构建产物：

```json
{
   "exports": {
      ".": {
         "types": "./src/index.ts",
         "development": "./src/index.ts",
         "import": "./dist/index.mjs"
      }
   },
   "publishConfig": {
      "exports": {
         ".": {
            "types": "./dist/index.d.mts",
            "import": "./dist/index.mjs"
         }
      }
   }
}
```

---

## pnpm-workspace.yaml 完整模板

```yaml
packages:
   - 'apps/*'
   - 'packages/*'
   - 'internal/*'

catalog:
   # ==================== 框架 ====================
   vue: ^3.5.0
   vue-router: ^4.5.0
   pinia: ^2.3.0

   # ==================== 构建工具 ====================
   vite: ^6.0.0
   tsup: ^8.0.0
   unbuild: ^3.0.0
   typescript: ^5.7.0

   # ==================== Vite 插件 ====================
   '@vitejs/plugin-vue': ^5.0.0

   # ==================== Lint ====================
   eslint: ^9.0.0
   prettier: ^3.0.0

   # ==================== 工程化 ====================
   turbo: ^2.0.0

   # ==================== 根据项目需要添加更多 ====================
   # axios: ^1.7.0
   # lodash-es: ^4.17.0
```

---

## turbo.json 完整模板

```json
{
   "$schema": "https://turbo.build/schema.json",
   "globalDependencies": ["tsconfig.json", "pnpm-workspace.yaml"],
   "tasks": {
      "build": {
         "dependsOn": ["^build"],
         "outputs": ["dist/**"],
         "cache": true
      },
      "dev": {
         "dependsOn": ["^build"],
         "cache": false,
         "persistent": true
      },
      "typecheck": {
         "dependsOn": ["^build"],
         "cache": true
      },
      "lint": {
         "cache": true
      },
      "clean": {
         "cache": false
      },
      "stub": {
         "cache": false
      }
   }
}
```
