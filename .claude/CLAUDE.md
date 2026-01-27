# CLAUDE.md

## 技术栈选型

**选型原则**

1. **现代化优先** - 选择生态活跃、维护良好的技术栈，确保长期支持
2. **开发效率** - 优先采用TypeScript、组件自动导入等减少重复工作的工具
3. **团队能力** - 基于团队现有技术栈，降低学习成本
4. **性能优化** - Vite、Pinia等工具提供更好的构建和运行时性能

| 领域           | 使用                                      | 不使用                     |
| -------------- | ----------------------------------------- | -------------------------- |
| 前端           | Vue 3 + Vite + Pinia + SCSS               | React、Webpack、Less、vuex |
| 语言           | TypeScript (必选)                         | JavaScript (新项目禁止)    |
| 后端           | NestJS + Prisma                           | Express、Koa、Sequelize    |
| 管理           | pnpm+Turborepo/workspaces(按需)           | Lerna、npm/yarn workspaces |
| 跨端           | Uniapp（按需）                            | Taro、React Native         |
| 样式           | SCSS + TailwindCSS（按需）                | Less、Stylus               |
| 组件库         | Ant Design Vue / Element Plus (按需)      | -                          |
| 组件自动导入   | unplugin-vue-components（组件库时必选）   | -                          |
| 开发工具       | Vue DevTools（Web、Electron 端）          | -                          |
| 路由           | Vue Router                                | -                          |
| 代码格式化     | prettier +cspell + eslint/stylelint(按需) | 其他格式化工具             |
| 前端网络请求   | axios                                     | fetch和其他网络请求工具    |
| 静态文档类项目 | vitepress                                 | 其他文档类框架             |

---

## 决策矩阵

当面临多种方案时，按以下优先级决策：

| 优先级 | 维度     | 权重 | 检查问题                 |
| ------ | -------- | ---- | ------------------------ |
| 1      | 一致性   | 高   | 是否符合项目现有模式？   |
| 2      | 简洁性   | 高   | 是否是最简单的解决方案？ |
| 3      | 可测试性 | 中   | 能否轻松编写测试？       |
| 4      | 可读性   | 中   | 6个月后还有人能看懂吗？  |
| 5      | 可逆性   | 低   | 未来改动成本是否可接受？ |

**典型决策场景**：

| 场景                 | 决策依据                            | 结果                      |
| -------------------- | ----------------------------------- | ------------------------- |
| 组件封装 vs 直接使用 | 一致性优先（看项目已有模式）        | 遵循现有模式              |
| 抽象函数 vs 复制代码 | 简洁性优先（<3次重复不抽象）        | 3次以上再抽象             |
| interface vs type    | 可读性优先（对象用interface）       | 对象→interface，联合→type |
| 新增依赖 vs 自行实现 | 简洁性 + 可逆性                     | 成熟库优先，简单逻辑自研  |
| 环境变量 vs 硬编码   | 安全性（禁止硬编码敏感信息）        | 必须用环境变量            |
| 是否用Turborepo      | 大型前端应用或多项目（前端+Nest等） | Turborepo                 |
| 是否用Uniapp         | 需要小程序/APP/H5跨端               | Uniapp（必选）            |

---

## TypeScript 规范

### 必须遵守

| 规则     | 要求                              |
| -------- | --------------------------------- |
| 严格模式 | tsconfig 必须 `strict: true`      |
| 禁止 any | 用 `unknown` 代替                 |
| 类型定义 | 对象用 `interface`，联合用 `type` |
| 类型导出 | 优先用 `export type`              |

### 命名规范

| 类型      | 规范                 | 示例                    |
| --------- | -------------------- | ----------------------- |
| 组件文件  | PascalCase.vue       | `UserProfile.vue`       |
| 文件夹    | kebab-case           | `user-profile/`         |
| 变量/函数 | camelCase            | `getUserInfo`           |
| 常量      | SCREAMING_SNAKE_CASE | `API_BASE_URL`          |
| 类/接口   | PascalCase           | `UserService`           |
| 接口前缀  | I + PascalCase       | `IUser`、`IUserService` |
| 类型别名  | PascalCase + Type    | `StatusType`            |
| 枚举      | PascalCase           | `UserRole`              |

### 禁止事项

```ts
// ❌ 禁止
const data: any = ...
function foo(data: any) { }
console.log('调试', data) // 禁止提交
export { IUser } from './types'
export type { IUser } from './types' // 混合导出

// ✅ 正确
const data: unknown = ...
function isString(val: unknown): val is string { ... }
logger.info('数据', data)
export type { IUser }
```

### 错误处理

所有 async/await 必须用 try/catch：

```ts
// ✅ 正确
async function foo() {
   try {
      const result = await apiCall()
      return result
   } catch (error) {
      handleError(error)
   }
}

// ❌ 错误
async function foo() {
   const result = await apiCall() // 未处理异常
}
```

### 注释规范

| 场景       | 要求                     |
| ---------- | ------------------------ |
| 复杂逻辑   | 必须注释                 |
| 公开 API   | 使用 JSDoc               |
| 无意义注释 | 禁止（如 `i++ // i加1`） |

```ts
/**
 * 获取用户信息
 * @param id - 用户 ID
 * @returns 用户对象
 */
async function getUser(id: string): Promise<IUser> {}
```

---

## Nestjs API规范

### URL 规则

| 规则        | 正确             | 错误            |
| ----------- | ---------------- | --------------- |
| 名词复数    | `/users`         | `/user`         |
| 小写+连字符 | `/user-profiles` | `/userProfiles` |
| 不超过 3 层 | `/users/1/posts` | `/a/b/c/d`      |
| 版本控制    | `/api/v1/users`  | `/api/users`    |

---

### 响应格式（统一）

```ts
interface ApiResponse<T = any> {
   code: number // 0 = 成功
   data: T
   message: string
   timestamp: number
}
```

---

## 前端 API 请求规范

### 核心规则

**所有网络请求必须放在 `src/api/` 目录，禁止在组件/composables 中直接发起请求**

### 代码示例

```ts
// ❌ 禁止：组件中直接请求
<script setup lang="ts">
const data = await fetch('/api/user')
const res = await axios.get('/api/user')
</script>

// ✅ 正确：通过 api 层
<script setup lang="ts">
import { getUserInfo } from '@/api/user'
const data = await getUserInfo()
</script>

// ❌ 禁止：composables 中直接请求
export function useUser() {
   fetch('/api/user').then(...)
}

// ✅ 正确：composables 调用 api 层
import { getUserInfo } from '@/api/user'
export function useUser() {
   getUserInfo().then(...)
}
```

### API 层示例

```ts
// src/api/user/types.ts
export interface IUser {
   id: string
   name: string
}

export interface GetUserParams {
   id: string
}

// src/api/user/index.ts
import { request } from '@/utils/request'
import type { IUser, GetUserParams } from './types'

export function getUserInfo(params: GetUserParams): Promise<IUser> {
   return request.get('/user/info', { params })
}
```

---

## 安全规范

### 环境变量

```bash
# .gitignore 必须包含
.env
.env.local
.env.*.local
```

```ts
// ✅ 正确
const apiUrl = import.meta.env.VITE_API_BASE_URL

// ❌ 错误
const apiUrl = 'https://api.example.com'
```

### Token 存储

| 位置            | 安全性 | 说明     |
| --------------- | ------ | -------- |
| localStorage    | ❌     | 易受 XSS |
| Pinia Memory    | ✅     | 推荐     |
| HttpOnly Cookie | ✅     | 后端设置 |

### Token 传递

```ts
// ✅ 正确：Header 传递
axios.defaults.headers.common['Authorization'] = `Bearer ${token}`

// ❌ 错误：URL 传递
axios.get('/api/user?token=xxx')
```

### XSS 防护

```vue
<!-- ❌ 危险 -->
<div v-html="userInput"></div>

<!-- ✅ 正确：DOMPurify -->
<script setup>
import DOMPurify from 'dompurify'
const safeHtml = computed(() => DOMPurify.sanitize(userInput.value))
</script>
<template>
   <div v-html="safeHtml"></div>
</template>
```

### 依赖安全

```bash
# 定期扫描
pnpm audit

# 锁定有漏洞的包
{
   "pnpm": {
      "overrides": {
         "lodash": "^4.17.21"
      }
   }
}
```

---

## CSpell 配置

```jsonc
{
   "allowCompoundWords": true
}
```

---

## 禁止事项汇总

| 行为                               | 原因            |
| ---------------------------------- | --------------- |
| 使用 `any` 类型                    | 破坏类型安全    |
| 组件中直接发起网络请求             | 难以维护、测试  |
| `console.log` 提交到代码           | 污染生产环境    |
| 提交 .env 文件                     | 泄露敏感信息    |
| 深层导入（`@/comp/Menu/Menu.vue`） | 破坏桶模式      |
| 使用 `require`                     | 必须用 `import` |
| 组件内联样式（非动态）             | 难以维护        |
| 混合导出类型                       | 导致类型错误    |

---

## vue文件模板（唯一标准）

```vue
<script setup lang="ts">
// 组件名自动使用文件名，无需 defineOptions
</script>

<template>
   <!-- 内容 -->
</template>

<style lang="scss" scoped>
/* 样式 */
</style>
```

**禁止**：

- `<script setup>` 不加 `lang="ts"`（TS 项目）
- `<style>` 不加 `scoped`
- 使用 `require`（必须用 `import`）
- 内联样式（非动态属性）

---

## vue大型组件（超过 1000 行）

```
components/
├── Menu/
│   ├── src/
│   │   ├── Menu.vue           # 主组件
│   │   ├── MenuItem.vue       # 子组件
│   │   ├── types.ts           # 类型定义
│   │   └── constants.ts       # 常量
│   └── index.ts               # export { default } from './src/Menu.vue'
```

---

## Barrel Pattern

每个组件/模块目录必须包含 `index.ts`：

```ts
// components/Menu/index.ts
export { default } from './Menu.vue'

// 使用
import Menu from '@/components/Menu'

// ❌ 禁止深层导入（< 50 行组件允许单文件，无需桶文件）
import Menu from '@/components/Menu/Menu.vue'
```

---

## 具体配置

### 前端axios配置

**参考文档**：[axios 封装](https://ttt1231.github.io/Turw-docs/frontdesign/common-problems.html#axios%E5%B0%81%E8%A3%85)

### 前端组件二次封装

```vue
<script setup lang="ts">
import { Button } from 'ant-design-vue'
import { getCurrentInstance, h, type ComponentPublicInstance } from 'vue'
import type { ButtonProps } from 'ant-design-vue'

/**
 * 组件的二次封装
 * 1、属性
 * 2、事件
 * 3、方法
 * 4、插槽
 * 5、类型
 * @example 自定义属性
 * interface CustomButtonProps extends ButtonProps {
 *  customProp?: string;
 * }
 * //do something with customProp
 */

type ButtonInstance = ComponentPublicInstance<ButtonProps>
const vm = getCurrentInstance()

//将事件方法暴露给父组件，供其ref（父组件）调用
function changeRef(expose: Element | ButtonInstance | null) {
   if (vm) {
      vm.exposed = expose
   }
}

//ts 提示
defineExpose({} as ButtonInstance)
</script>
<template>
   <!-- 注意这里不能用div或者容器包裹，否则事件会冒泡会被重复执行 -->
   <component :is="h(Button, { ...$attrs, ref: changeRef }, $slots)"> </component>
</template>
```

### unplugin-vue-components配置

```ts
// vite.config.ts
import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'
// import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default {
   plugins: [
      Components({
         resolvers: [
            AntDesignVueResolver({ importStyle: false })
            // ElementPlusResolver(),
         ]
      })
   ]
}
```

```gitignore
# 自动生成的类型声明
components.d.ts
```

**TypeScript 配置**：需要在 `tsconfig.app.json` 中添加类型声明

```json
// tsconfig.app.json
{
   "compilerOptions": {
      "types": [
         "ant-design-vue/typings/global.d.ts"
         // "element-plus/global"
      ]
   }
}
```

**说明**：添加后组件类型才能被识别（如 `AButton`、`AMenu` 等带前缀的类型）

### stylelint配置参考

[配置参考文档](https://ttt1231.github.io/Turw-docs/code-style/eslint-format.html#%E9%85%8D%E7%BD%AE-stylelint)

### vitepress项目配置

@projects/vitepress.md

### env验证

**参考文档**：[env 配置验证注意](https://ttt1231.github.io/Turw-docs/frontdesign/Vite.html#env%E9%85%8D%E7%BD%AE%E9%AA%8C%E8%AF%81%E6%B3%A8%E6%84%8F)
