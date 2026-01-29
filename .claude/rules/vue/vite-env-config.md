---
paths:
   - '**/vite*.config.{ts,js}'
---

# Vite 配置文件环境变量加载

```ts
export default defineConfig(({ mode }) => {
   const env = loadEnv(mode, process.cwd())

   return {
      server: {
         port: Number(env.VITE_PORT) || 5173
      }
   }
})
```

## 注意事项

- ❌ 能不用process.env就不用，除非明确要求
- ✅ 环境变量值都是 `string` 类型，使用时注意类型转换
- ✅ 只有 `VITE_` 前缀的变量才会暴露给客户端代码
