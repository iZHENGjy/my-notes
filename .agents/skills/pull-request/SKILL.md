---
name: pull-request
description: Create a new feature branch, commit changes, push to GitHub, and open a pull request — all in one command. Use when the user asks to open a PR, create a pull request, or push changes for review. / 一条命令搞定：建 feature 分支、提交改动、推到 GitHub、开 PR。当用户要"开 PR"、"create pull request"、"推改动求审"时触发。
---

# Pull Request 命令

一条命令搞定整个 PR 流程：建分支、提交、推到 GitHub、用合适描述开 PR。贡献功能或修复时用最合适。

## 任务

把整个 pull request 工作流自动化：建分支、暂存改动、用描述性消息提交、推到 GitHub、开带正确描述的 PR。

## 流程

### 1. **检查前置**

- 确认是 git 仓库
- 检查要包含的未提交改动
- 验证 GitHub CLI（`gh`）可用
- 把当前分支当 base 分支
- 如果已经在 feature 分支上，问："从当前分支创建 PR？"

### 2. **建 Feature 分支**

```bash
# 从 PR 标题生成分支名，或用提供的名字
# 清洗分支名：小写、空格换连字符、去特殊字符
branch_name=$(echo "$branch_name" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g' | sed 's/--*/-/g')

# 检查分支是否已存在
if git show-ref --verify --quiet refs/heads/$branch_name; then
  echo "Branch $branch_name already exists, using alternative name"
  branch_name="${branch_name}-$(date +%s)"
fi

# 格式：feature/short-description 或 fix/issue-name
git checkout -b $branch_name
```

### 3. **暂存并审查改动**

- 给用户看 `git status`
- 给用户看 `git diff --staged`
- 没暂存改动就 `git add -A` 暂存全部
- 继续前和用户确认改动

### 4. **提交改动**

- 分析改动，写有意义的 commit 信息
- 用 conventional commits 格式（feat:、fix:、docs: 等）
- 改动复杂时加详细 commit body

```bash
git commit -m "feat: add new feature

- 细节 1
- 细节 2

🤖 Generated with Claude Code"
```

### 5. **推到 GitHub**

```bash
# 推并设置 upstream tracking
git push -u origin feature/[branch-name]
```

### 6. **创建 Pull Request**

用 `gh pr create`，配上：

- 描述性标题
- 详细 body：
  - 改动摘要
  - 测试 checklist
  - 相关 issue（如果有）
- 设置 base 分支（一般是 main / master）

```bash
gh pr create \
  --title "Feature: Add awesome new capability" \
  --body "$(cat <<'EOF'
## Summary
简要描述这个 PR 做了什么

## Changes
- 加了 X 功能
- 修了 Y bug
- 改进了 Z 的性能

## Testing
- [ ] 本地测过
- [ ] 所有测试通过
- [ ] 文档更新

## Screenshots
（如果适用）

🤖 Generated with [Claude Code](https://claude.ai/code)
EOF
)" \
  --base main
```

### 7. **给出后续步骤**

- 显示 PR URL
- 提醒审查流程
- 建议下一步动作（请人审、加标签等）

## 参数

- **可选**：分支名（不给就从改动自动生成）
- **可选**：PR 标题（不给就从改动分析）
- **可选**：目标分支（默认 main / master）

## 用法示例

```bash
# 从改动自动生成分支和 PR
/pull-request

# 指定分支名
/pull-request feature/add-auth

# 完整指定
/pull-request fix/bug-123 "Fix: Resolve authentication timeout issue" develop
```

## 输出示例

```
📝 分析改动...
🌿 创建分支：feature/add-download-command
✅ 已提交：feat: add download-attachment command
📤 已推到 origin
🔗 PR 已创建：https://github.com/user/repo/pull/42

下一步：
- 请团队成员审查
- 加合适的标签
- 链接相关 issue
```

## 分支命名规范

- **新功能**：`feature/description`
- **修复**：`fix/issue-or-description`
- **文档**：`docs/what-updated`
- **重构**：`refactor/what-changed`
- **性能**：`perf/optimization`
- **测试**：`test/what-tested`

## Commit 信息格式

遵循 conventional commits：

- `feat:` 新功能
- `fix:` Bug 修复
- `docs:` 仅文档
- `style:` 格式化、缺分号等
- `refactor:` 不修 bug 也不加功能的代码改动
- `perf:` 性能改进
- `test:` 加缺失测试
- `chore:` 构建流程或辅助工具变更

## 安全特性

- 改动很大时 push 前确认
- commit 前给出 diff
- 创建前验证 PR 描述
- 检查该分支是否已有 PR
- 优雅处理 merge 冲突

## 错误处理

- 没改动："No changes to create PR"
- 已经在 feature 分支：问要不要从当前分支建 PR
- PR 已存在：显示已有 PR URL
- push 失败：检查权限和 remote 设置
