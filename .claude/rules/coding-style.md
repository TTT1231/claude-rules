---
paths:
  - '**/*.{ts,js,vue}'
---

# 代码风格

## 不可变性（核心）
始终创建新对象，禁止直接修改：

```javascript
// ❌ 错误：直接修改
function updateUser(user, name) {
   user.name = name // 发生修改！
   return user
}

// ✅ 正确：保持不可变
function updateUser(user, name) {
   return {
      ...user,
      name
   }
}
```

## 文件组织
多小文件 > 少大文件：
- 高内聚、低耦合
- 通常 200-400 行，最多 800 行
- 从大组件中抽取工具函数
- 按功能/领域组织，而非按类型

## 错误处理
必须完整处理错误：

```typescript
try {
   const result = await riskyOperation()
   return result
} catch (error) {
   console.error('操作失败:', error)
   throw new Error('详细且用户友好的错误信息')
}
```

## 输入校验
必须校验用户输入：

```typescript
import { z } from 'zod'

const schema = z.object({
   email: z.string().email(),
   age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## 代码质量检查清单
工作完成前确认：
- [ ] 代码可读、命名清晰
- [ ] 函数简短（<50 行）
- [ ] 文件聚焦（<800 行）
- [ ] 无深层嵌套（>4 层）
- [ ] 正确处理错误
- [ ] 无 console.log 语句
- [ ] 无硬编码值
- [ ] 无直接修改（使用不可变模式）
