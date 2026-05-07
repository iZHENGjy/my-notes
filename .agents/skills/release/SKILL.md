---
name: release
description: Automatically bump version, update changelog, commit, tag, and push a new release based on recent changes. Use when the user wants to cut a release, publish a new version, bump the version, or run the release workflow. / 基于最近改动自动 bump 版本、更新 changelog、commit、打 tag、push 新 release。当用户要"发版"、"发布新版本"、"bump version"、跑 release 工作流时触发。
allowed-tools: [Read, Write, Edit, MultiEdit, Bash, Grep]
---

# Release 命令

把整个发布流程自动化：分析最近 commit 决定版本 bump 类型、更新 package.json 里的版本、把 unreleased changelog 条目挪到新版本段、提交全部、打 git tag、推到 GitHub。

## 任务

1. 分析自上次 tag 以来的 commit，决定版本 bump 类型
2. 更新 package.json 里的版本
3. 把 CHANGELOG.md 里 "Unreleased" 段挪到新版本段
4. 提交改动
5. 创建带注解的 git tag
6. 把 commit 和 tag 推到 GitHub

## 流程

1. **检查前置**
   - 确保在 main / master 分支
   - 检查未提交改动
   - 确认 CHANGELOG.md 和 package.json 存在
   - 从 package.json 取当前版本

2. **决定版本 bump**
   - 如果给了参数（major / minor / patch），用那个
   - 否则分析自上次 tag 以来的 commit：
     - 看到 "BREAKING CHANGE" 或 "!" = major bump
     - 看到 "feat:" = minor bump
     - 看到 "fix:"、"docs:"、"chore:" = patch bump
   - 算出新版本号

3. **更新文件**
   - 更新 package.json 里的版本
   - 把 CHANGELOG.md 的 "Unreleased" 段挪到新版本段
   - 加新版本的对比链接
   - 创建新的空 "Unreleased" 段

4. **Git 操作**
   - 暂存改动：`git add package.json CHANGELOG.md`
   - 提交：`git commit -m "chore: release v{version}"`
   - 创建带注解 tag：`git tag -a v{version} -m "Release v{version}"`
   - 推 commit：`git push`
   - 推 tag：`git push --tags`

5. **创建 GitHub Release**
   - 用 `gh release create` 自动发布 release
   - 从 CHANGELOG.md 抽取该版本段作为 release notes
   - 加上 "Generated with Claude Code" 落款
   - 这样 release 在 GitHub releases 页可见

6. **给确认**
   - 显示 GitHub release URL
   - 确认成功发布

## 版本 bump 规则

### 语义化版本（MAJOR.MINOR.PATCH）

**快速决策**：

- 用户能做以前做不到的事？→ **MINOR**
- 之前能用的东西坏了？→ **MAJOR**（破坏性）或 **PATCH**（修复）
- 之前能用的东西变好了？→ **PATCH**

**MAJOR**（1.0.0 → 2.0.0）：

- 破坏性变更，要求用户改代码 / 配置
- 移除功能或命令
- 不兼容地改命令语法或行为
- commit body 含 "BREAKING CHANGE"
- type 后带 "!" 的 commit（如 "feat!:"）

**MINOR**（1.0.0 → 1.1.0）：

- 加了**新能力**（不是已有功能的增强）
- 让以前做不到的事变得可能
- 新命令、新工具、新集成
- 不影响已有功能的新可选特性
- 启用新功能的重大架构变更
- 以 "feat:" 开头、加新功能的 commit
- 例子：
  - 加新 `/command`
  - 加新 MCP server
  - 加 vault import 能力（首次）
  - 让 upgrade 在没 git 连接时也能工作（之前不能）
  - 让一个之前需要联网的功能能离线工作

**PATCH**（1.0.0 → 1.0.1）：

- bug 修复和小改进
- 已有功能的增强（已经能用的）
- 性能改进
- 文档更新
- 不改行为的重构
- "fix:"、"docs:"、"style:"、"refactor:"、"perf:"、"test:"、"chore:" 的 commit
- 例子：
  - 让已有命令更聪明（但不开新用例）
  - 改善错误信息
  - 修已有功能的 bug
  - 增强已有 import，更智能
  - 改善已有功能的 UI / 格式

### Commit 信息最佳实践

**只在新功能用 "feat:"**：

- ✅ `feat: add vault import capability`
- ❌ `feat: enhance vault import`（应该 `fix:` 或 `refactor:`）

**改进和修正用 "fix:"**：

- ✅ `fix: improve vault detection accuracy`
- ✅ `fix: correct file counting in init-bootstrap`

**代码改进用 "refactor:"**：

- ✅ `refactor: enhance profile building with URL fetching`
- ✅ `refactor: make init-bootstrap questions smarter`

**性能改进用 "perf:"**：

- ✅ `perf: optimize vault analysis for large vaults`

## 用法示例

```bash
# 从 commit 自动检测版本 bump
claude run release

# 强制指定版本 bump
claude run release patch
claude run release minor
claude run release major

# 输出示例：
# 📦 当前版本：0.1.0
# 🔍 分析自上次 release 以来的 commit...
#
# 找到 commit：
# - feat: add video support to Gemini Vision
# - docs: update README with setup instructions
# - fix: correct attachment link handling
#
# ✨ 检测到版本 bump：MINOR（新功能）
# 📝 新版本：0.2.0
#
# ✅ 已更新 package.json
# ✅ 已更新 CHANGELOG.md
# ✅ 已提交改动
# ✅ 已创建 tag v0.2.0
# ✅ 已推到 GitHub
# ✅ 已创建 GitHub release
#
# 🎉 Release v0.2.0 完成！
#
# GitHub Release: https://github.com/user/repo/releases/tag/v0.2.0
```

## 错误处理

- 不在 main 分支："Please switch to main branch first"
- 有未提交改动："Please commit or stash changes first"
- 自上次 release 以来无改动："No changes to release"
- 版本已存在："Version X.X.X already exists"

## 安全特性

- Dry run 模式：显示要发生什么，不实际改动
- push 前确认提示
- 校验版本号格式
- 创建前检查已有 tag
