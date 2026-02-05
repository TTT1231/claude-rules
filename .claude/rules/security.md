# Security Guidelines

## Mandatory Security Checks

Must complete before any commit:

- [ ] No hardcoded sensitive information (API keys, passwords, tokens)
- [ ] All user input is validated
- [ ] SQL injection protection (parameterized queries)
- [ ] XSS protection (HTML sanitization)
- [ ] CSRF protection enabled
- [ ] Authentication/authorization verified
- [ ] Rate limiting enabled on all endpoints
- [ ] Error messages do not leak sensitive data

## Sensitive Information Management

```typescript
// Prohibited: Hardcoded sensitive information
const apiKey = 'sk-proj-xxxxx'

// Required: Use environment variables
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
   throw new Error('OPENAI_API_KEY not configured')
}
```

## Security Response Protocol

When security issues are discovered:

1. Stop operations immediately
2. Use **security-reviewer** agent
3. Fix critical issues before proceeding
4. Rotate all leaked sensitive information
5. Review entire codebase for similar issues
