---
paths:
   - '**/*.vue'
   - '**/*.ts'
   - '**/*.d.ts'
   - '**/*.js'
   - '**/*.mjs'
   - '**/*.cjs'
   - '**/*.json'
   - '**/*.jsonc'
---

# 文件组织规范

---

## Barrel Pattern（桶文件模式）

多文件目录应包含 `index.ts` 桶文件统一导出。

### 使用条件

**推荐使用 index.ts**（满足任一）：

- 目录内 `.ts` 文件数 ≥ 3（推荐阈值，可根据项目实际情况调整）
- 目录需要被外部模块导入
- NestJS 模块目录

**无需 index.ts**：

- 单文件工具/函数/类型
- 仅内部使用的私有目录

### 导出方式规则

```ts
// ✅ export * 用于：同级别文件、类型定义、常量
export * from './types'
export * from './constants'
export * from './utils'

// ✅ 命名导出用于：需要重命名、选择性导出、默认导出转命名
export { default as UsersModule } from './users.module'
export { HttpClient } from './client' // 只从 client.ts 中选择性地导出某些内容

// ❌ 禁止：对同一文件同时使用 export * 和命名导出
export * from './client'
export { SomethingElse } from './client' // 重复，混淆
```

### 目录结构示例

```
utils/
├── http/
│   ├── request.ts
│   ├── response.ts
│   ├── client.ts
│   ├── types.ts
│   └── index.ts          // export * from './types'; export * from './request'; export * from './response'; export { HttpClient } from './client'
├── string/
│   └── format.ts         // 单文件，无需 index.ts
└── index.ts              // export * from './http'; export * from './string'

// 使用
import { HttpClient, formatString } from '@/utils'
```

### NestJS 模块示例

```
users/
├── users.module.ts
├── users.controller.ts
├── users.service.ts
├── dto/
│   ├── create-user.dto.ts
│   ├── update-user.dto.ts
│   └── index.ts          // export * from './create-user.dto'; export * from './update-user.dto'
├── entities/
│   └── user.entity.ts     // 单文件，无需 index.ts
└── index.ts               // export { default as UsersModule } from './users.module'; export * from './dto'; export * from './entities'

// 使用
import { UsersModule } from '@/modules/users'
import { CreateUserDto } from '@/modules/users'
```

### 原则

对外提供统一入口，简化导入路径，便于内部重构。

---

## 多文件组件（Vue）

> **边界说明**：本节专门针对 Vue 组件的目录组织。其他框架（如 React）的组件组织方式不在此范围内。

适用于需要拆分的 Vue 组件目录组织。

### 拆分标准

**唯一标准：复杂度**

| 条件         | 说明                     | 示例                         |
| ------------ | ------------------------ | ---------------------------- |
| **多职责**   | 组件承担多个不相关的功能 | 同时处理数据展示、编辑、导出 |
| **深层嵌套** | 模板嵌套层级 > 4 层      | div 里套 div 再套 div...     |
| **复用需求** | 子逻辑可能被其他地方复用 | 弹窗、表单项、列表项         |
| **类型导出** | 需要导出类型供外部使用   | 复杂的 Props/Emits 类型      |

> **行数不是拆分依据**：一个 200 行但逻辑复杂的组件应该拆分，而一个 600 行但结构简单的列表组件可以保留。

### 单文件 vs 多文件选择

| 场景                                               | 推荐方式 | 目录结构                               | 导入方式                                           |
| -------------------------------------------------- | -------- | -------------------------------------- | -------------------------------------------------- |
| 简单组件（单一职责、浅层嵌套、无复用子组件）       | 单文件   | `components/UserCard.vue`              | `import UserCard from '@/components/UserCard.vue'` |
| 复杂组件（多职责、深层嵌套、有子组件、需导出类型） | 多文件   | `components/Menu/src/...` + `index.ts` | `import { Menu } from '@/components/Menu'`         |

### 目录结构（使用 Barrel Pattern）

```
Menu/
├── src/
│   ├── Menu.vue           # 主组件（必须默认导出）
│   ├── MenuItem.vue       # 子组件
│   ├── types.ts           # 类型定义
│   └── constants.ts       # 常量
└── index.ts               # 桶文件
```

```ts
// Menu/index.ts
export { default as Menu } from './src/Menu.vue'
export { default as MenuItem } from './src/MenuItem.vue'
export * from './src/types'
export * from './src/constants'

// 使用
import { Menu, MenuItem, IMenu } from '@/components/Menu'

// ❌ 避免：深层导入
import Menu from '@/components/Menu/src/Menu.vue'
```

**说明**：单文件组件允许直接导入，多文件组件（有 `src/` 目录）应通过桶文件导入。

---

## Monorepo 组织规范

> **适用范围**：使用 pnpm workspace + Turborepo 的 monorepo 架构

### 架构分层

```
packages/
├── base/              # 底层基础包（零依赖）
│   ├── @scope/utils/
│   ├── @scope/typings/
│   └── @scope/composables/
├── lib/               # 中层领域包（可组合底层）
│   ├── ui/
│   ├── api/
│   └── auth/
└── app/               # 上层应用包（专注业务）
    ├── web/
    ├── admin/
    └── mobile/
```

**层级约束**：

- **base 层**：零依赖，不能引入任何 workspace 依赖
- **lib 层**：可依赖 base 层，同层包尽量不互相依赖
- **app 层**：可依赖 base 和 lib 层，只能依赖下层

### 包命名规范

**作用域包名格式**：`@scope/domain`

| 层级 | 示例                                                   | 说明                              |
| ---- | ------------------------------------------------------ | --------------------------------- |
| base | `@scope/utils`, `@scope/typings`, `@scope/composables` | 目录在 `packages/base/@scope/xxx` |
| lib  | `@scope/ui`, `@scope/api`, `@scope/auth`               | 目录在 `packages/lib/xxx`         |
| app  | `@scope/web`, `@scope/admin`                           | 目录在 `packages/app/xxx`         |

**项目根 internal 目录**：

- 位置：`internal/`（非 packages/ 下）
- 用途：存放项目配置包，如 `eslint-config`, `cspell-config`
- 约定：其他子包不使用 internal 目录

### package.json exports 配置

**必须配置**：所有子包都需要 `exports` 字段 + `index.ts` 桶文件

**职责划分**：

- `exports`：定义公共 API 入口，控制外部可访问路径
- `index.ts`：组织包内模块聚合导出，作为代码层面入口

**配置粒度原则**：

- **底层包（base 层）**：细粒度子路径导出，按需引用
- **上层包（lib/app 层）**：粗粒度聚合导出，简化使用

#### 底层包配置示例

```json
// packages/base/@scope/utils/package.json
{
   "name": "@scope/utils",
   "exports": {
      ".": "./src/index.ts",
      "./string": "./src/string/index.ts",
      "./http": "./src/http/index.ts",
      "./format": "./src/format/index.ts"
   }
}
```

```ts
// packages/base/@scope/utils/src/index.ts
// 主入口导出最常用的工具
export { formatDate, parseDate } from './format'
export { HttpClient } from './http'

// 子路径入口
// packages/base/@scope/utils/src/string/index.ts
export { toCamelCase, toSnakeCase } from './case'
export { trim, split } from './basic'

// 使用（按需引入，避免全量引入）
import { toCamelCase } from '@scope/utils/string'
import { HttpClient } from '@scope/utils/http'
```

#### 上层包配置示例

```json
// packages/lib/ui/package.json
{
   "name": "@scope/ui",
   "exports": {
      ".": "./src/index.ts",
      "./types": "./src/types.ts"
   }
}
```

```ts
// packages/lib/ui/src/index.ts
// 粗粒度聚合导出，简化使用
export * from './components'
export * from './composables'
export * from './directives'

// 使用
import { Button, Modal } from '@scope/ui'
```

### 依赖管理

#### 依赖声明格式

```json
// 依赖内部包：使用 workspace 协议
{
  "dependencies": {
    "@scope/utils": "workspace:*",
    "@scope/ui": "workspace:*"
  },
  "devDependencies": {
    "@scope/eslint-config": "workspace:*"
  }
}

// 外部包：正常版本号
{
  "dependencies": {
    "vue": "^3.4.0",
    "axios": "^1.6.0"
  }
}
```

#### 跨包导入方式

```ts
// ✅ 正确：使用作用域包名导入
import { Button } from '@scope/ui'
import { formatDate } from '@scope/utils/format'

// ❌ 错误：使用相对路径
import { Button } from '../../packages/lib/ui/src/components/Button'
```

#### 循环依赖处理

**治本原则**：通过架构约束避免循环依赖

| 场景     | 约束                                    |
| -------- | --------------------------------------- |
| base 层  | **零依赖**，不能引入任何 workspace 依赖 |
| 同层包   | 尽量不互相依赖，非必须不用              |
| 必要互引 | 直接引用，通过设计减少互引用            |

**检测工具**：使用 `madge` 等工具检测循环依赖

### 类型定义三层架构

```
packages/
├── base/@scope/typings/     # 第1层：底层原子类型
├── lib/types/               # 第2层：业务聚合
└── lib/ui/form/             # 第3层：领域特定（与组件一起导出）
```

#### 第1层：底层原子类型

```ts
// packages/base/@scope/typings/src/index.ts
export type * from './basic'
export type * from './user'
export type * from './menu'
```

#### 第2层：业务聚合

```ts
// packages/lib/types/src/index.ts
// 聚合底层类型 + 本地业务类型
export type * from '@scope/typings'
export type * from './business'
```

#### 第3层：领域特定

```ts
// packages/lib/ui/form/src/types.ts
// 与组件一起导出，不放入公共 types
export interface FormItemProps { ... }

// packages/lib/ui/form/index.ts
export { Form, FormItem } from './components'
export type * from './types'
```

### 内部模块隔离

**规则**：只有项目根级 `internal/` 目录存放配置包，其他子包不使用 internal

**公共 API 控制**：通过 `exports` 字段显式声明可访问路径

```json
// packages/lib/ui/package.json
{
   "exports": {
      ".": "./src/index.ts",
      "./types": "./src/types.ts"
      // ❌ 不暴露 internal/
   }
}
```

### 目录结构模板

**灵活结构**：根据包类型灵活调整

```
# 工具包（base 层）
packages/base/@scope/utils/
├── src/
│   ├── string/
│   │   └── index.ts
│   ├── http/
│   │   └── index.ts
│   └── index.ts
├── package.json
└── tsconfig.json

# UI 组件库（lib 层）
packages/lib/ui/
├── src/
│   ├── components/
│   ├── composables/
│   ├── types.ts
│   └── index.ts
├── package.json
└── tsconfig.json

# 业务应用（app 层）
packages/app/web/
├── src/
│   ├── pages/
│   ├── composables/
│   └── main.ts
├── package.json
└── vite.config.ts
```

### 本地调试

**依赖配置**：使用 `workspace:*` 协议，pnpm 自动链接

```json
{
   "dependencies": {
      "@scope/ui": "workspace:*" // pnpm 自动链接本地包
   }
}
```

**开发流程**：

1. 修改本地包代码
2. 依赖该包的应用自动使用最新版本（无需重新链接）
3. Turborepo 按需重新构建

### 导入最佳实践

```ts
// ✅ 推荐：按需导入底层工具
import { toCamelCase } from '@scope/utils/string'
import { formatDate } from '@scope/utils/format'

// ❌ 避免：全量导入底层工具
import * as Utils from '@scope/utils'

// ✅ 推荐：导入上层聚合包
import { Button, Modal, Form } from '@scope/ui'

// ✅ 推荐：导入类型
import type { User } from '@scope/typings'

// ❌ 避免：深层路径导入
import Button from '@scope/ui/src/components/Button/Button.vue'

// ❌ 避免：同层包互引用（非必要）
// packages/lib/auth 引用 packages/lib/user
// 应考虑抽取公共逻辑到 base 层
```

---
