# Vue Web 项目配置

本文档适用于 Vue 3 + Vite 的 Web 应用项目配置。

---

## unplugin-vue-components 配置

当使用组件库（Ant Design Vue / Element Plus）时，应配置 `unplugin-vue-components` 实现组件自动导入。

### 配置示例

```typescript
// vite.config.ts
import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
   plugins: [
      vue(),
      Components({
         resolvers: [
            AntDesignVueResolver({
               importStyle: false // css in js
            })
         ]
      })
   ]
})
```

### 注意事项

1. **components.d.ts 文件管理**

   插件会在项目根目录自动生成 `components.d.ts` 类型声明文件，**必须将其加入 Git 版本控制**。

2. **组件库类型声明**

   在 `tsconfig.json` 中添加组件库的全局类型声明，否则类型（如 `AButton`、`AMenu`）无法被识别：

   ```json
   {
      "compilerOptions": {
         "types": [
            "ant-design-vue/typings/global.d.ts" // Ant Design Vue
            // 或 "element-plus/global"           // Element Plus
         ]
      }
   }
   ```

3. **按需导入 vs 全量导入**

   | 方式                | 优点       | 缺点       |
   | ------------------- | ---------- | ---------- |
   | 按需导入 (unplugin) | 打包体积小 | 需要配置   |
   | 全量导入            | 配置简单   | 打包体积大 |

   **推荐**：生产环境使用按需导入。

---

## Vite 配置建议

### 路径别名

```typescript
// vite.config.ts
export default defineConfig({
   resolve: {
      alias: {
         '@': path.resolve(__dirname, 'src')
      }
   }
})
```

配套 `tsconfig.json`：

```json
{
   "compilerOptions": {
      "paths": {
         "@/*": ["src/*"]
      }
   }
}
```

### 环境变量

```typescript
// .env.development
VITE_API_BASE_URL=http://localhost:3000

// .env.production
VITE_API_BASE_URL=https://api.example.com
```

使用时：

```typescript
const apiUrl = import.meta.env.VITE_API_BASE_URL
```

> 注意：Vite 环境变量必须以 `VITE_` 开头才能暴露给客户端代码。
