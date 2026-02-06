# 诊断检查项完整清单

本文档定义了 Monorepo 诊断模式的所有检查项，包括检查算法、判定标准和修复模板。

---

## 检查项总览

| #   | 检查项                | 严重程度 | 默认启用 | 前置条件          |
| --- | --------------------- | -------- | -------- | ----------------- |
| 1   | 循环依赖检测          | 🔴 错误  | ✅       | —                 |
| 2   | 层级违规检测          | 🔴 错误  | ✅       | —                 |
| 3   | 幽灵依赖检测          | 🔴 错误  | ✅       | —                 |
| 4   | catalog 一致性检查    | 🟡 警告  | ✅       | 使用 pnpm catalog |
| 5   | workspace 引用检查    | 🔴 错误  | ✅       | —                 |
| 6   | exports 配置审查      | 🟡 警告  | ✅       | —                 |
| 7   | stub 配置检查         | 🟡 警告  | ❌       | 使用 unbuild      |
| 8   | turbo.json 审查       | 🟡 警告  | ✅       | 使用 Turborepo    |
| 9   | tsconfig extends 检查 | 🟡 警告  | ✅       | —                 |
| 10  | 包命名规范检查        | 🔵 建议  | ✅       | —                 |
| 11  | private 标记检查      | 🟡 警告  | ✅       | —                 |
| 12  | 跨包类型安全检查      | 🟡 警告  | ✅       | —                 |

> **智能预选规则**：扫描项目后根据实际使用的工具链自动调整默认启用状态。例如未安装 unbuild 则 #7 默认不选中；无 turbo.json 则 #8 默认不选中。

---

## 检查项详解

### 1. 循环依赖检测

**目标**：检测 workspace 包之间是否存在循环依赖

**算法**：

```
1. 遍历所有子包的 package.json
2. 提取 dependencies 和 devDependencies 中以 workspace: 开头的依赖
3. 构建有向图 G：节点 = 包名，边 = 依赖关系
4. DFS 遍历 G，检测是否存在回边（back edge）
5. 如存在回边，记录完整的循环路径
```

**判定标准**：

- 🔴 存在循环依赖 → 错误

**输出示例**：

```
🔴 循环依赖：@myproject/utils → @myproject/types → @myproject/utils
   📍 packages/utils/package.json (dependencies: "@myproject/types": "workspace:*")
   📍 packages/types/package.json (dependencies: "@myproject/utils": "workspace:*")
   💡 分析两个包的实际依赖关系，将共享部分提取到新包或移除不必要的依赖
```

**修复模板**：

- 提取共享代码到新的底层包
- 或将一方的依赖方向反转
- 或合并两个高度耦合的包

---

### 2. 层级违规检测

**目标**：检测包之间的引用是否违反三层分层规则

**算法**：

```
1. 确定每个包的层级：
   - 匹配 apps/* → 应用层
   - 匹配 packages/* → 共享包层
   - 匹配 internal/* → 工程工具层
2. 遍历每个包的 package.json：
   a. 检查 dependencies 中的 workspace:* 引用（运行时依赖）
   b. 检查 devDependencies 中的 workspace:* 引用
3. 对 dependencies 中的引用，严格执行层级规则矩阵
4. 对 devDependencies 中的引用：
   - 如果引用的是 internal 包，检查是否为配置包（tsconfig/eslint-config 等）→ 允许
   - 如果引用的是 apps 包 → 始终违规
5. （可选）扫描 src/ 目录下的 import 语句，交叉验证
```

**判定标准**：

- 🔴 dependencies 中存在层级违规 → 错误
- 🟡 devDependencies 中存在非配置类的层级违规 → 警告

**输出示例**：

```
🔴 层级违规：apps/frontend 的 dependencies 引用了 internal 包
   📍 apps/frontend/package.json:12
   → "@myproject/vite-config": "workspace:*" 在 dependencies 中
   💡 如果是配置引用，应移到 devDependencies；如果是运行时引用，应从 internal 移到 packages
```

---

### 3. 幽灵依赖检测

**目标**：检测源码中 import 的包是否在当前子包的 package.json 中声明

**算法**：

```
1. 遍历每个子包的 src/ 目录下的 .ts/.vue/.tsx 文件
2. 提取所有 import/require 语句中的模块名
3. 筛选出第三方模块（非相对路径、非 node 内置模块）
4. 检查该模块是否在当前包的 dependencies 或 devDependencies 中声明
5. 如果未声明但在根 package.json 或其他包中存在 → 幽灵依赖
```

**判定标准**：

- 🔴 import 了未在当前包 package.json 中声明的第三方依赖 → 错误

**输出示例**：

```
🔴 幽灵依赖：packages/utils 使用了 lodash-es 但未在 package.json 中声明
   📍 packages/utils/src/helpers.ts:3 → import { cloneDeep } from 'lodash-es'
   💡 在 packages/utils/package.json 的 dependencies 中添加 "lodash-es": "catalog:"
```

**修复模板**：

```json
// 在对应包的 package.json dependencies 中添加
"lodash-es": "catalog:"
```

---

### 4. catalog 一致性检查

**目标**：验证 pnpm catalog 的使用一致性

**算法**：

```
1. 读取 pnpm-workspace.yaml 中的 catalog 定义
2. 遍历所有子包的 package.json：
   a. 检查使用 catalog: 协议的依赖是否在 catalog 中有定义
   b. 检查第三方依赖是否使用了硬编码版本而非 catalog:
   c. 收集所有实际使用的 catalog 条目
3. 对比 catalog 定义和实际使用：
   - 有定义但未被任何包使用的 → 冗余条目
   - 使用了 catalog: 但 catalog 中未定义的 → 缺失定义
   - 同一第三方依赖在某些包用 catalog: 其他包用硬编码版本 → 不一致
```

**判定标准**：

- 🔴 catalog: 引用但 catalog 中未定义 → 错误
- 🟡 同一依赖混用 catalog: 和硬编码版本 → 警告
- 🔵 catalog 中有未使用的条目 → 建议

**输出示例**：

```
🟡 catalog 不一致：axios 在 apps/frontend 中使用 "catalog:" 但在 apps/backend 中硬编码为 "^1.7.0"
   💡 统一使用 catalog: 协议，将版本声明移到 pnpm-workspace.yaml 的 catalog 中

🔵 catalog 冗余条目：react（定义在 catalog 中但未被任何包使用）
   💡 如不再需要，从 pnpm-workspace.yaml 的 catalog 中移除
```

---

### 5. workspace 引用检查

**目标**：验证 workspace:\* 引用的包是否实际存在

**算法**：

```
1. 收集 workspace 中所有包的包名（从各 package.json 的 name 字段）
2. 遍历所有子包，提取 workspace:* 依赖引用的包名
3. 检查引用的包名是否在 workspace 包名列表中
```

**判定标准**：

- 🔴 引用的包在 workspace 中不存在 → 错误

**输出示例**：

```
🔴 workspace 引用失效：apps/frontend 引用了 @myproject/auth 但该包不存在
   📍 apps/frontend/package.json:15
   💡 检查包名是否拼写正确，或创建该包
```

---

### 6. exports 配置审查

**目标**：检查每个非 private 包的 exports 字段是否正确配置

**算法**：

```
1. 遍历 packages/ 下的所有子包（apps 和 internal 跳过）
2. 检查 package.json 是否有 exports 字段
3. 如有 exports，检查每个导出路径：
   a. 是否包含 types 条件（TypeScript 项目必须）
   b. 是否包含 import 条件
   c. types 指向的文件是否存在
   d. import 指向的文件路径是否合理（dist/ 目录）
   e. （可选）是否有 development 条件导出（推荐）
4. 检查 main/module/types 顶层字段与 exports 是否一致
```

**判定标准**：

- 🟡 缺少 exports 字段 → 警告
- 🟡 exports 中缺少 types 条件 → 警告
- 🔵 缺少 development 条件导出 → 建议

**输出示例**：

```
🟡 exports 不完整：packages/utils 缺少 types 条件
   📍 packages/utils/package.json
   💡 在 exports 中添加 "types": "./src/index.ts"（开发时）或 "./dist/index.d.ts"（发布时）
```

**修复模板**：

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

---

### 7. stub 配置检查

**前置条件**：项目使用了 unbuild

**目标**：检查 unbuild 的 stub 开发模式是否正确配置

**算法**：

```
1. 确认项目使用了 unbuild（根或子包 devDependencies 中有 unbuild）
2. 遍历使用 unbuild 的包：
   a. 检查是否有 stub 脚本（"stub": "unbuild --stub"）
   b. 检查是否有 postinstall 自动 stub（"postinstall": "pnpm run stub" 或类似）
3. 检查根 package.json 是否有统一的 stub 或 postinstall 脚本
```

**判定标准**：

- 🟡 使用 unbuild 但缺少 stub 脚本 → 警告
- 🔵 缺少 postinstall 自动 stub → 建议

**输出示例**：

```
🟡 缺少 stub 脚本：internal/vite-config 使用 unbuild 但没有 stub 脚本
   💡 在 package.json scripts 中添加 "stub": "unbuild --stub"

🔵 建议添加统一的 postinstall stub：
   💡 在根 package.json 中添加 "postinstall": "pnpm -r run stub --if-present"
      或在各包中添加 "postinstall": "pnpm run stub"
```

---

### 8. turbo.json 审查

**前置条件**：项目使用了 Turborepo

**目标**：审查 turbo.json 配置的完整性和正确性

**算法**：

```
1. 读取 turbo.json
2. 检查 task 定义：
   a. build task 是否有 dependsOn: ["^build"]（确保拓扑构建顺序）
   b. build task 是否有 outputs: ["dist/**"]（缓存产物）
   c. dev task 是否设置 cache: false 和 persistent: true
   d. typecheck task 是否有 dependsOn
3. 检查是否有 globalDependencies（建议追踪根级配置文件变化）
4. 检查子包的 package.json 中的 scripts：
   - 有 turbo task 但子包缺少对应 script → 可能导致 turbo 跳过
```

**判定标准**：

- 🟡 build task 缺少 dependsOn: ["^build"] → 警告（可能导致构建顺序错误）
- 🟡 build task 缺少 outputs → 警告（无法缓存）
- 🔵 缺少 globalDependencies → 建议

**输出示例**：

```
🟡 turbo.json：build task 缺少 dependsOn: ["^build"]
   💡 添加 "dependsOn": ["^build"] 确保依赖包先构建

🔵 turbo.json：建议添加 globalDependencies
   💡 添加 "globalDependencies": ["tsconfig.json", "pnpm-workspace.yaml"] 追踪全局配置变化
```

---

### 9. tsconfig extends 检查

**目标**：验证各子包的 tsconfig 是否正确继承共享配置

**算法**：

```
1. 查找 internal/ 下的 tsconfig 配置包
2. 遍历所有子包的 tsconfig.json：
   a. 检查是否有 extends 字段
   b. 如有 extends，检查引用的配置文件是否存在
   c. 检查 extends 的值是否使用项目统一的 tsconfig 包
3. 检查 tsconfig 包的 devDependencies 引用是否正确
```

**判定标准**：

- 🟡 子包的 tsconfig 未继承共享配置 → 警告
- 🔴 extends 引用的配置文件不存在 → 错误

**输出示例**：

```
🟡 tsconfig 未继承：packages/utils/tsconfig.json 未使用共享 tsconfig
   💡 添加 "extends": "@myproject/tsconfig/lib.json"
```

---

### 10. 包命名规范检查

**目标**：检查包名和目录名的命名规范

**算法**：

```
1. 收集所有包的名称和目录路径
2. 检查命名空间一致性：
   a. packages/ 下的包是否使用统一的 @scope/
   b. internal/ 下的包是否使用统一的 @scope/
3. 检查目录名拼写：
   a. internal 不应拼写为 internel 或其他变体
   b. 目录名应使用 kebab-case
4. 检查包名与目录名是否匹配
```

**判定标准**：

- 🔵 命名空间不统一 → 建议
- 🟡 目录名拼写异常（如 internel） → 警告
- 🔵 包名与目录名不匹配 → 建议

**输出示例**：

```
🟡 目录名拼写异常：internel/ 疑似 internal/ 的拼写错误
   💡 将 internel/ 重命名为 internal/，并更新所有引用路径

🔵 命名空间不一致：packages 下混用 @test/ 和 @mymonorepo/ 命名空间
   💡 建议统一使用同一个命名空间
```

---

### 11. private 标记检查

**目标**：检查 apps/ 和 internal/ 下的包是否标记了 private: true

**算法**：

```
1. 遍历 apps/ 下所有包的 package.json
   - 检查是否有 "private": true
2. 遍历 internal/ 下所有包的 package.json
   - 检查是否有 "private": true
```

**判定标准**：

- 🟡 apps/ 下的包缺少 private: true → 警告
- 🟡 internal/ 下的包缺少 private: true → 警告

**输出示例**：

```
🟡 缺少 private 标记：internal/tsconfig 未标记 private: true
   📍 internal/tsconfig/package.json
   💡 添加 "private": true 防止意外发布
```

---

### 12. 跨包类型安全检查

**目标**：检测语义层面的不合理跨包引用

**算法**：

```
1. 识别前端组件包：packages/ 下包含 .vue 文件或依赖 vue 的包
2. 识别后端应用：apps/ 下的 NestJS/Express 等后端应用
3. 检查后端应用是否引用了前端组件包
4. 检查前端组件包是否引用了 Node.js 专用包
```

**判定标准**：

- 🟡 后端应用引用了前端组件包 → 警告
- 🟡 前端组件包引用了 Node.js 专用包 → 警告

**输出示例**：

```
🟡 跨包类型违规：apps/backend-nestjs 引用了 @myproject/components（前端组件包）
   📍 apps/backend-nestjs/package.json:18
   💡 后端应用不应引用前端组件包，请检查是否为误操作
```

---

## 诊断报告模板

```markdown
## 🔍 Monorepo 诊断报告

**项目**：{project_path}
**扫描时间**：{timestamp}
**子包数量**：{total_packages} 个（apps: {apps_count}, packages: {packages_count}, internal: {internal_count}）

### 检查结果总览

| #   | 检查项       | 状态     | 问题数  |
| --- | ------------ | -------- | ------- |
| 1   | 循环依赖检测 | {status} | {count} |
| 2   | 层级违规检测 | {status} | {count} |
| ... | ...          | ...      | ...     |

**总计**：🔴 {errors} 个错误 / 🟡 {warnings} 个警告 / 🔵 {suggestions} 个建议

---

### 🔴 错误（必须修复）

{error_details}

### 🟡 警告（建议修复）

{warning_details}

### 🔵 建议（可选优化）

{suggestion_details}

---

> 使用 `/monorepo fix` 可自动修复上述部分问题。
```
