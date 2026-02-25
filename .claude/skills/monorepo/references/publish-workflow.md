# 版本发布与 Changelog 工作流 (Changesets)

在 Monorepo 中，管理多个包的版本号和生成 Changelog 是一项复杂的任务。推荐使用 [Changesets](https://github.com/changesets/changesets) 来处理。

---

## 核心概念

1. **Changeset**：一个包含变更描述和版本提升类型（major/minor/patch）的 Markdown 文件。
2. **Version**：消耗所有的 Changesets，自动计算并更新相关包的 `package.json` 版本号，并生成 `CHANGELOG.md`。
3. **Publish**：将更新后的包发布到 npm 注册表。

---

## 初始化配置

### 1. 安装依赖

在根目录安装：

```bash
pnpm add -wD @changesets/cli
```

### 2. 初始化

```bash
pnpm changeset init
```

这会在根目录生成 `.changeset/config.json`。

### 3. 配置 `.changeset/config.json`

```json
{
   "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
   "changelog": "@changesets/cli/changelog",
   "commit": false,
   "fixed": [],
   "linked": [],
   "access": "public",
   "baseBranch": "main",
   "updateInternalDependencies": "patch",
   "ignore": ["apps/*", "internal/*"]
}
```

**关键配置说明**：

- `ignore`：忽略不需要发布的包（如应用层和内部配置层）。
- `access`：发布到 npm 时的访问权限（`public` 或 `restricted`）。
- `updateInternalDependencies`：当依赖的内部包版本更新时，当前包如何更新依赖版本。

---

## 工作流步骤

### 1. 开发与提交变更 (开发者)

开发者在完成一个功能或修复一个 Bug 后，运行：

```bash
pnpm changeset
```

CLI 会引导开发者：

1. 选择哪些包发生了变更。
2. 选择版本提升类型（major, minor, patch）。
3. 输入变更描述（将出现在 Changelog 中）。

这会在 `.changeset` 目录下生成一个随机命名的 Markdown 文件。开发者将此文件与代码一起提交。

### 2. 版本发布 (维护者/CI)

当准备发布新版本时，运行：

```bash
pnpm changeset version
```

此命令会：

1. 读取所有未消耗的 Changesets。
2. 计算每个包的新版本号（处理依赖图，如果 A 依赖 B，B 更新了，A 也会相应更新）。
3. 更新各包的 `package.json`。
4. 更新各包的 `CHANGELOG.md`。
5. 删除已消耗的 Changesets 文件。

维护者检查变更无误后，提交这些更改：

```bash
git commit -am "chore: release new versions"
```

### 3. 发布到 npm

```bash
pnpm build # 确保所有包已构建
pnpm changeset publish
```

此命令会将所有版本号有更新的包发布到 npm。

---

## CI/CD 集成 (GitHub Actions)

推荐使用 Changesets 官方的 GitHub Action 实现自动化发布。

`.github/workflows/release.yml`:

```yaml
name: Release

on:
   push:
      branches:
         - main

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
   release:
      name: Release
      runs-on: ubuntu-latest
      steps:
         - name: Checkout Repo
           uses: actions/checkout@v4

         - name: Setup pnpm
           uses: pnpm/action-setup@v3
           with:
              version: 9

         - name: Setup Node.js
           uses: actions/setup-node@v4
           with:
              node-version: 20
              cache: 'pnpm'

         - name: Install Dependencies
           run: pnpm install --frozen-lockfile

         - name: Create Release Pull Request or Publish
           id: changesets
           uses: changesets/action@v1
           with:
              publish: pnpm release
              commit: 'chore: release new versions'
              title: 'chore: release new versions'
           env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
              NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

在根 `package.json` 中添加 `release` 脚本：

```json
{
   "scripts": {
      "release": "pnpm build && changeset publish"
   }
}
```
