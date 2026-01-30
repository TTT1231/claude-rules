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

## CSpell 配置

```jsonc
{
   "allowCompoundWords": true
}
```

---

## 禁止事项汇总

| 行为           | 原因         | 详见                                  |
| -------------- | ------------ | ------------------------------------- |
| 提交 .env 文件 | 泄露敏感信息 | [安全指南](.claude/rules/security.md) |
| 其他规范       | -            | 见各专项规则文件                      |

> 注：详细禁止事项请查看 [TypeScript 规范](.claude/rules/typescript-rule.md)、[Vue 规范](.claude/rules/vue/vue-rule.md)

---

## 具体配置

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

---
