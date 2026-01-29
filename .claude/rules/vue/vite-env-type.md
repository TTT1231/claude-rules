---
paths:
   - '**/*.d.ts'
---

# Vite 环境变量类型安全

## 规则

在 Vite 项目中，使用以下方式定义 `import.meta.env` 的类型：

```typescript
/// <reference types="vite/client" />

declare global {
   interface ImportMetaEnv extends EnvConfig {}

   interface ImportMeta {
      readonly env: ImportMetaEnv
   }
}

export {}
```

**说明：**

- `EnvConfig` 从全局类型中导入，需在项目的全局类型文件中定义
- 使用 `declare global` 使类型声明全局生效
- 末尾的 `export {}` 将该文件视为模块，确保全局声明正确扩展

## EnvConfig 定义示例

在项目的全局类型文件中（如 `src/types/env.ts`）定义：

```typescript
// src/types/env.ts
interface EnvConfig {
   readonly VITE_API_BASE_URL: string
   readonly VITE_APP_TITLE: string
   // ... 其他环境变量
}
```

然后在 `tsconfig.json` 中确保该文件被包含：

```json
// tsconfig.json
{
   "include": ["src/**/*.d.ts", "env.d.ts", "src/types/**/*.ts"]
}
```

或使用 Vite 推荐配置（在 `tsconfig.app.json` 中）**最推荐**：

```json
// tsconfig.app.json
{
   "include": ["**/*.d.ts", "src/types/**/*.ts"]
}
```

## 禁止

```typescript
// ❌ 禁止：直接在 .d.ts 文件中定义 EnvConfig
/// <reference types="vite/client" />

interface EnvConfig {
   readonly VITE_API_BASE_URL: string
}

declare global {
   interface ImportMetaEnv extends EnvConfig {}
}

export {}

// ❌ 禁止：不使用 declare global
interface ImportMetaEnv {
   readonly VITE_API_BASE_URL: string
}

// ❌ 禁止：忘记 export {}
/// <reference types="vite/client" />

declare global {
   interface ImportMetaEnv extends EnvConfig {}
}
// 缺少 export {}
```
