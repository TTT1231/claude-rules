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

### env验证

**参考文档**：[env 配置验证注意](https://ttt1231.github.io/Turw-docs/frontdesign/Vite.html#env%E9%85%8D%E7%BD%AE%E9%AA%8C%E8%AF%81%E6%B3%A8%E6%84%8F)

---
