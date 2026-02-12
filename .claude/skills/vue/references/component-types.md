# 组件分类与拆分规则

## 组件分类

| 类型       | 特征                          | 示例                            |
| ---------- | ----------------------------- | ------------------------------- |
| **展示型** | 纯 UI、无状态、数据来自 props | UserCard、StatusBadge           |
| **容器型** | 含状态/逻辑、组合多个组件     | UserList、Dashboard             |
| **功能型** | 复用逻辑、无 UI               | useTable、useForm（Composable） |

---

## 大组件拆分

### 拆分条件

- **主要**：业务逻辑复杂（3+ 个独立业务逻辑、混合多种状态管理）
- **辅助**：代码行数 > 1300 行

### 拆分策略

```
按 UI 区域拆分（第一层）
  ↓
按关注点分离（第二层）
```

### 目录结构示例

```
UserTable/
├── UserTable.vue            # 主组件
├── UserTableHeader.vue      # 表头
├── UserTableBody.vue        # 表体
├── UserTableFilters.vue     # 筛选
└── composables/
    ├── useTableData.ts      # 数据逻辑
    └── useTableFilters.ts   # 筛选逻辑
```

### 拆分原则

1. **UI 优先**：先按视觉区域划分，再考虑逻辑拆分
2. **数据下沉**：数据获取逻辑放在最底层的组件或 Composable
3. **事件上浮**：用户交互事件向上传递到业务处理层
4. **接口清晰**：子组件通过 props/emits 与父组件通信
