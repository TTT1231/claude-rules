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

## web-vue插件配置注意事项

### unplugin-vue-components配置注意事项

当它被配置的时候，注意它会自动在根目录生成一个`components.d.ts`文件，需要将它加入到Git中，同时如果启用了组件库，需要开启对这个插件对组件库类型的支持，也就是在`tsconfig`配置文件中添加类型声明即可:`ant-design-vue/typings/global.d.ts`或者`element-plus/global`，添加后组件类型才能被识别（如 `AButton`、`AMenu` 等带前缀的类型）。

---
