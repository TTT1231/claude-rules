---
paths:
   - 'docs/.vitepress/config.{mts,mjs,ts}'
---

# VitePress 文档项目配置

```ts
// .vitepress/config.ts
export default defineConfig({
   // Sass: 使用现代 API
   css: {
      preprocessorOptions: {
         scss: {
            api: 'modern-compiler'
         }
      }
   },

   // 代码块行号（按需）
   markdown: {
      lineNumbers: true
   },

   // 语言默认中文
   locales: {
      root: { label: '简体中文', lang: 'zh-CN' }
   },

   // 显示最后更新时间（按需）
   lastUpdated: true,

   // 文档搜索（默认本地搜索，特殊需求按需配置）
   themeConfig: {
      search: {
         provider: 'local'
      }
   }
})
```

**自定义容器示例**：

```markdown
:::tip 提示内容
:::

:::warning 警告内容
:::

> [!NOTE] GitHub 风格提示
> [!WARNING] GitHub 风格警告
```
