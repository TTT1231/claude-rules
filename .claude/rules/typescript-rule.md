---
paths:
   - '**/*.ts'
   - '**/*.d.ts'
   - '**/*.vue'
---

# TypeScript 规范

## 生效范围

**文件类型**：`*.ts`、`*.d.ts`、`*.vue`

**对于 `.vue` 文件**：本规范仅适用于 `<script lang="ts">` 内的代码，不涉及 `template` 和 `style`。

---

## 必须遵守

| 规则     | 要求                                                        |
| -------- | ----------------------------------------------------------- |
| 严格模式 | tsconfig 必须 `strict: true`                                |
| 禁止 any | 用 `unknown` 代替                                           |
| 类型定义 | `interface` 用于可扩展对象，`type` 用于联合、交叉、工具类型 |
| 类型导出 | 类型用 `export type`，值用 `export`                         |

---

## 命名规范

| 类型      | 规范       | 示例                  |
| --------- | ---------- | --------------------- |
| 变量/函数 | camelCase  | `getUserInfo`         |
| 常量      | UPPER_CASE | `API_BASE_URL`        |
| 类/接口   | PascalCase | `UserService`、`User` |
| 类型别名  | PascalCase | `UserStatus`          |
| 枚举      | PascalCase | `UserRole`            |
| 文件夹    | kebab-case | `user-service/`       |

---

## 禁止事项

| 行为               | 原因                       |
| ------------------ | -------------------------- |
| 使用 `any` 类型    | 破坏类型安全               |
| 同一标识符混合导出 | 导致编译失败（重复标识符） |
| `console.log` 提交 | 污染生产环境               |
| 使用 `require`     | 必须用 `import`            |

```ts
// ❌ 禁止
const data: any = ...
function foo(data: any) { }

// ❌ 禁止：同一标识符混合导出（编译失败：Duplicate identifier）
export { IUser } from './types'
export type { IUser } from './types'

// ✅ 正确
const data: unknown = ...
function isString(val: unknown): val is string { ... }

// ✅ 正确：同一标识符只使用一种导出方式
export type { IUser } from './types'              // 类型只用 export type
export { UserService } from './service'           // 值只用 export
```

---

## 错误处理

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

---

## 注释规范

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

**Better Comments 特殊标记：**

```ts
//! 重要信息：需要特别关注的内容
//? 疑问/解释：用于解释复杂逻辑或标记待确认问题
// TODO 待办：标记需要完成的任务
//!、?、TODO 也可以在 JSDoc 中使用，例如：
/**
 * //! 重要说明
 * @param xxx
 */
```

**非空断言 `!` 与可选链 `?`：**

```ts
// ❌ 避免：无理由的非空断言
const name = user!.name

// ✅ 正确：使用可选链
const name = user?.name

// ❌ 错误：缺乏运行时检查的断言
function getValue(): string | null {
   return Math.random() > 0.5 ? 'value' : null
}
const value = getValue()! // 危险！实际可能返回 null

// ✅ 正确：先进行运行时检查，再断言
function getValue(): string | null {
   return 'value'
}
const value = getValue()! // 安全：函数实现保证永不返回 null

// ✅ 更佳：配合类型守卫使用
function getValue(): string | null {
   return 'value'
}
const value = getValue()
if (!value) {
   throw new Error('value is required')
}
// 此处 value 已被类型守卫收窄为 string，无需 !
```

**使用原则**：非空断言仅在能从代码中明确看出值不可能为 null/undefined 时使用。

## .d.ts 文件特殊规范

- 只包含类型定义，不包含实现逻辑
- 必须使用 `export` 或 `import` 确保是模块
- 全局类型扩展使用 `declare global`

```ts
// ✅ 正确：.d.ts 文件示例
export interface User {
   id: string
   name: string
}

// ✅ 正确：全局类型扩展
declare global {
   interface Window {
      myCustomProperty: string
   }
}
export {} // 必须有：将文件变为模块，使 declare global 生效

// ❌ 禁止：包含实现逻辑
export function getUser() {
   return fetch('/api/user') // 不应该在 .d.ts 中实现
}
```

---

## 泛型

### 命名规范

| 场景     | 命名规则               | 示例                         |
| -------- | ---------------------- | ---------------------------- |
| 单个泛型 | `T`                    | `function identity<T>(x: T)` |
| 多个泛型 | 描述性名称             | `<TKey, TValue>`、`<TProps>` |
| 泛型约束 | `T extends Constraint` | `<T extends object>`         |
| 组件泛型 | `TProps`               | `defineProps<TProps>()`      |

### 使用规则

```ts
// ✅ 正确：单个泛型用 T
function identity<T>(value: T): T {
   return value
}

// ✅ 正确：多个泛型用描述性名称
function map<TKey, TValue>(
   obj: Record<TKey, TValue>,
   fn: (value: TValue, key: TKey) => void
): void {
   // ...
}

// ✅ 正确：提供默认值
function fetchData<T = unknown>(url: string): Promise<T> {
   // ...
}

// ✅ 正确：使用约束代替运行时检查
function getProperty<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
   return obj[key]
}

// ❌ 避免：过度泛化
// 如果只有 1-2 种使用场景，可能不需要泛型
function process<T>(data: T): T {
   // 没有实际使用 T 的约束或操作，不如直接用具体类型
   return data
}
```

### 原则

- 泛型参数 ≤ 2 个时，保持简洁
- 泛型参数 ≥ 3 个时，考虑用配置对象代替
- 优先使用约束而非内部类型判断

---

## 类型守卫

### 自定义类型守卫

```ts
// ✅ 正确：使用 is 关键字
function isString(val: unknown): val is string {
   return typeof val === 'string'
}

function isNotNull<T>(val: T | null): val is T {
   return val !== null
}

// ✅ 正确：使用守卫收窄类型
function process(value: unknown) {
   if (isString(value)) {
      // 此处 value 类型为 string
      return value.toUpperCase()
   }
}

// ❌ 错误：返回 boolean 而非类型谓词
function isStringBad(val: unknown): boolean {
   return typeof val === 'string'
}
// 使用后仍需类型断言，失去守卫意义
```

### 命名规范

| 场景     | 命名模式          | 示例                           |
| -------- | ----------------- | ------------------------------ |
| 类型检查 | `is[Type]`        | `isString`、`isNumber`         |
| 非空检查 | `isNotNull`       | `isNotNull`、`isDefined`       |
| 结构验证 | `isValid[Entity]` | `isValidUser`、`isValidConfig` |

### 原则

- 优先使用内置 `typeof`、`instanceof`
- 自定义守卫必须有明确的类型收窄效果
- 守卫函数不应有副作用

---

## 工具类型

### 优先使用内置工具类型

```ts
// ✅ 正确：使用内置工具类型
type PartialUser = Partial<User>
type UserWithoutPassword = Omit<User, 'password'>
type UserKeys = keyof User
type UserRoles = Pick<User, 'roles'>

// ✅ 正确：组合使用
type UpdateUserDto = Partial<Pick<User, 'name' | 'email'>>

// ❌ 禁止：重复造轮片
// 不要自己实现内置类型
type MyPartial<T> = {
   [P in keyof T]?: T[P]
}
```

### 常用工具类型场景

| 工具类型       | 使用场景                    |
| -------------- | --------------------------- |
| `Partial<T>`   | 更新 DTO、可选配置          |
| `Required<T>`  | 必填校验                    |
| `Pick<T, K>`   | 提取部分字段                |
| `Omit<T, K>`   | 排除敏感字段（如 password） |
| `Record<K, T>` | 字典类型、枚举映射          |
| `Readonly<T>`  | 不可变数据                  |

---

## 条件类型

### 使用规则

```ts
// ✅ 正确：简单条件类型
type NonNullable<T> = T extends null | undefined ? never : T

// ✅ 正确：使用 infer 推断
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T
type UnwrapArray<T> = T extends (infer U)[] ? U : T

// ❌ 避免：过度嵌套（超过 2 层）
type Complex<T> = T extends A ? X : T extends B ? Y : T extends C ? Z : never
// 拆分为多个类型别名提高可读性
```

### 原则

- 超过 2 层嵌套时，拆分为多个类型别名
- 优先用 `infer` 推断而非手动指定
- 复杂逻辑应配合 JSDoc 说明意图

---

## 类型推断

### 避免冗余类型注解

```ts
// ❌ 冗余：类型显而易见
const count: number = 1
const name: string = 'hello'
const items: number[] = [1, 2, 3]

// ✅ 正确：让 TypeScript 推断
const count = 1
const name = 'hello'
const items = [1, 2, 3]

// ✅ 正确：复杂对象显式注解（提高可读性）
const config: ServerConfig = {
   host: 'localhost',
   port: 8080,
   ssl: true
}

// ✅ 正确：函数返回值优先推断
function getUserId() {
   return user.id // 推断为 string | number
}

// ✅ 必要时显式声明返回值
function getUserId(): string {
   return user.id // 明确返回类型，便于重构
}
```

### 必须显式注解的场景

- 公开 API 的函数参数和返回值
- 复杂的内联对象类型
- 需要明确类型约束的泛型
- 导出的常量和变量

---

## 声明合并

### interface vs type

```ts
// ✅ interface 可以合并（用于扩展）
interface User {
   id: string
}

interface User {
   name: string
}

// 最终 User 包含 id 和 name

// ❌ type 不能合并
type Animal = {
   name: string
}

// Error: Duplicate identifier 'Animal'
type Animal = {
   age: number
}
```

### 使用场景

| 场景           | 推荐方式    | 原因                  |
| -------------- | ----------- | --------------------- |
| 扩展第三方类型 | `interface` | 支持声明合并          |
| 定义对象类型   | `interface` | 可扩展、更好 IDE 支持 |
| 联合/交叉类型  | `type`      | interface 不支持      |
| 工具类型       | `type`      | 需要使用泛型工具语法  |

### 扩展第三方类型

```ts
// ✅ 正确：扩展第三方库类型
declare module 'vue' {
   interface ComponentCustomProperties {
      $http: HttpClient
   }
}

// ✅ 正确：使用 declare global 扩展全局类型
declare global {
   interface Window {
      __APP_CONFIG__: AppConfig
   }
}
export {} // 必须有：将文件变为模块
```

### 原则

- 优先用 `type`，除非需要声明合并
- 同一文件内避免多次声明同一 interface
- 扩展第三方类型使用 `declare module` 或 `declare global`

---
