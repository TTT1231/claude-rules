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

---

## 状态管理 (Pinia)

推荐使用 Setup Store 语法（类似 Composition API）。

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
   // State
   const userInfo = ref<User | null>(null)
   const token = ref<string>('')

   // Getters
   const isLoggedIn = computed(() => !!token.value)

   // Actions
   const login = async (credentials: LoginDto) => {
      const res = await apiLogin(credentials)
      token.value = res.token
      userInfo.value = res.user
   }

   const logout = () => {
      token.value = ''
      userInfo.value = null
   }

   return { userInfo, token, isLoggedIn, login, logout }
})
```

---

## 路由配置 (Vue Router)

推荐使用路由懒加载（Lazy Loading）以优化首屏加载速度。

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
   history: createWebHistory(import.meta.env.BASE_URL),
   routes: [
      {
         path: '/',
         name: 'home',
         component: () => import('@/views/HomeView.vue') // ✅ 路由懒加载
      },
      {
         path: '/about',
         name: 'about',
         component: () => import('@/views/AboutView.vue')
      }
   ]
})

// 路由守卫示例
router.beforeEach((to, from, next) => {
   const userStore = useUserStore()
   if (to.meta.requiresAuth && !userStore.isLoggedIn) {
      next({ name: 'login', query: { redirect: to.fullPath } })
   } else {
      next()
   }
})

export default router
```

---

## API 请求封装 (Axios)

统一封装 Axios 实例，处理请求拦截、响应拦截和错误提示。

```typescript
// utils/request.ts
import axios from 'axios'
import { useUserStore } from '@/stores/user'
import { ElMessage } from 'element-plus'

const request = axios.create({
   baseURL: import.meta.env.VITE_API_BASE_URL,
   timeout: 10000
})

// 请求拦截器
request.interceptors.request.use(
   (config) => {
      const userStore = useUserStore()
      if (userStore.token) {
         config.headers.Authorization = `Bearer ${userStore.token}`
      }
      return config
   },
   (error) => Promise.reject(error)
)

// 响应拦截器
request.interceptors.response.use(
   (response) => {
      const res = response.data
      // 根据后端约定的状态码判断
      if (res.code !== 200) {
         ElMessage.error(res.message || '请求失败')
         return Promise.reject(new Error(res.message || 'Error'))
      }
      return res.data
   },
   (error) => {
      if (error.response?.status === 401) {
         const userStore = useUserStore()
         userStore.logout()
         // 跳转登录页...
      }
      ElMessage.error(error.message || '网络错误')
      return Promise.reject(error)
   }
)

export default request
```
