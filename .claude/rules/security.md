# 安全指南

## 强制性安全检查

任何提交前必须完成：

- [ ] 无硬编码敏感信息（API 密钥、密码、令牌）
- [ ] 所有用户输入已验证
- [ ] SQL 注入防护（参数化查询）
- [ ] XSS 防护（HTML 清理）
- [ ] CSRF 保护已启用
- [ ] 身份验证/授权已验证
- [ ] 所有端点已启用速率限制
- [ ] 错误消息不泄露敏感数据

## 敏感信息管理

```typescript
// 禁止：硬编码敏感信息
const apiKey = 'sk-proj-xxxxx'

// 必须：使用环境变量
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
   throw new Error('OPENAI_API_KEY not configured')
}
```

## 安全响应协议

发现安全问题时：

1. 立即停止操作
2. 使用 **security-reviewer** 代理
3. 在继续之前修复严重问题
4. 轮换所有已泄露的敏感信息
5. 审查整个代码库中是否存在类似问题
