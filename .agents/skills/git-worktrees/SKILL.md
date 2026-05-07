---
name: git-worktrees
description: Work with git worktrees for isolated parallel development. Use when starting feature work in isolation, when need separate workspace without branch switching, or when cleaning up worktrees after PR merge. / 用 git worktree 做隔离的并行开发。当要在隔离环境里开新 feature、需要不切分支的独立工作区、或 PR 合并后清理 worktree 时触发。
---

# Git Worktrees

## 概述

Git worktree 在共享同一个仓库的前提下创建隔离的工作区，让你能同时在多个分支上工作而不用切换。每个 worktree 是一个独立目录，有自己的 working tree，但共享同一个 `.git` 历史。

## 什么时候用

- **并行开发**：A 功能 build / test 时同时改 B 功能
- **代码审查**：检出 PR 分支，不打断当前工作
- **实验**：试一些有风险的东西，不影响主工作区
- **长时间任务**：feature 开发期间保持 main 分支可用

## 速查

| 操作 | 命令 |
|--------|---------|
| 列出 worktree | `git worktree list` |
| 创建 worktree | `git worktree add <path> -b <branch>` |
| 从已有分支创建 | `git worktree add <path> <branch>` |
| 移除 worktree | `git worktree remove <path>` |
| 清理过期 worktree | `git worktree prune` |

## 创建 Worktree

### 新建 feature 分支

```bash
# 创建 worktree + 新分支
git worktree add .worktrees/my-feature -b feat/my-feature

# 或指定 base 分支
git worktree add .worktrees/my-feature -b feat/my-feature main
```

### 从已有分支

```bash
# 检出已有的远程分支
git worktree add .worktrees/pr-review origin/fix-bug

# 检出已有本地分支
git worktree add .worktrees/hotfix hotfix/urgent-fix
```

## 目录结构

```
project/
├── .git/                    # 共享 git 历史
├── .worktrees/              # 约定：worktree 都放这
│   ├── feature-a/           # 第一个 worktree
│   └── feature-b/           # 第二个 worktree
└── src/                     # 主 worktree 文件
```

## 创建后设置

创建 worktree 后通常要做：

```bash
cd .worktrees/my-feature

# 装依赖
npm install  # 或 pnpm install、yarn 等

# 复制必要的 env 文件
cp ../.env .env.local

# 验证设置
npm test
```

## 安全规则

**绝不在没确认的情况下移除带未提交改动的 worktree。**

```bash
# 先检查未提交改动
git -C .worktrees/my-feature status --porcelain

# 空的就可以安全移除
git worktree remove .worktrees/my-feature

# 合并后删除分支（-d 是安全的，没合并会失败）
git branch -d feat/my-feature
```

### 移除决策矩阵

| PR 已合并？ | 有未提交改动？ | 行动 |
|------------|---------------------|--------|
| 是 | 否 | 可以安全移除 |
| 是 | 是 | 问用户 — 改动会丢 |
| 否 | 否 | **不要**移除 — 工作没保留 |
| 否 | 是 | **不要**移除 — 活跃工作 |

## 清理 Worktree

### 手动清理

```bash
# 1. 检查工作是否已合并（用 GitHub 的话）
gh pr list --head feat/my-feature --state merged

# 2. 检查未提交改动
git -C .worktrees/my-feature status --porcelain

# 3. 移除 worktree（仅在已合并或用户确认时）
git worktree remove .worktrees/my-feature

# 4. 删除分支
git branch -d feat/my-feature
```

### 清理过期 Worktree

如果 worktree 目录被手动删了：

```bash
git worktree prune
```

## 常见模式

### 审查 PR

```bash
# 从 PR 分支创建 worktree
git fetch origin pull/123/head:pr-123
git worktree add .worktrees/pr-123 pr-123

# 审查、测试，然后清理
git worktree remove .worktrees/pr-123
git branch -D pr-123
```

### 并行 feature 开发

```bash
# 主工作在项目根目录继续
# 在 worktree 里开新 feature
git worktree add .worktrees/new-api -b feat/new-api

# 同时干两件事
code .worktrees/new-api  # 打开新的 VS Code 窗口
```

## 排错

### "Branch already checked out"

一个分支同一时刻只能在一个 worktree 里检出：

```bash
# 找出分支在哪检出
git worktree list

# 先移除那个 worktree，或换个分支
```

### "Worktree directory not empty"

```bash
# 强制添加（如果目录存在但不是 worktree）
git worktree add --force <path> <branch>
```

### Worktree 被锁定

如果 worktree 被锁了（防误删）：

```bash
# 解锁
git worktree unlock <path>

# 再移除
git worktree remove <path>
```
