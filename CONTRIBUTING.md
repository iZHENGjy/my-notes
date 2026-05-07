# 贡献指南 — Claudesidian

感谢你想给 claudesidian 做贡献！本文档是给项目做贡献的规范。

## 开发环境

1. Fork 仓库
2. Clone 你的 fork：`git clone https://github.com/yourusername/claudesidian.git`
3. 装依赖：`pnpm install`
4. 建 feature 分支：`git checkout -b feature/your-feature-name`

## Commit 信息规范

我们用 [Conventional Commits](https://www.conventionalcommits.org/)，让 commit 历史清楚：

- `feat:` 新功能
- `fix:` Bug 修复
- `docs:` 文档变更
- `style:` 代码风格（格式化等）
- `refactor:` 重构
- `test:` 测试增删
- `chore:` 维护性任务

例：

```
feat: add new research-assistant command
fix: correct attachment link updates in scripts
docs: update README with MCP setup instructions
```

## 版本号

用 [Semantic Versioning](https://semver.org/)：

- MAJOR (1.0.0)：破坏性变更
- MINOR (0.1.0)：新功能（向后兼容）
- PATCH (0.0.1)：Bug 修复（向后兼容）

## Pull Request 流程

1. 在 CHANGELOG.md 的 "Unreleased" 段写上你的变更
2. 必要时更新文档
3. 确保所有脚本还能跑
4. 提 PR，写清楚改了什么

## CHANGELOG 更新

贡献时把变更加到 CHANGELOG.md 的 "Unreleased" 段：

```markdown
## [Unreleased]

### Added

- 你的新功能

### Fixed

- 你的 bug 修复
```

用这些分类：

- **Added** — 新功能
- **Changed** — 已有功能的变更
- **Deprecated** — 准备移除的功能
- **Removed** — 已移除的功能
- **Fixed** — Bug 修复
- **Security** — 安全更新

## 发布流程（维护者用）

1. 更新 package.json 里的版本号
2. 把 CHANGELOG.md 里 "Unreleased" 的内容挪到新版本段
3. Commit：`git commit -m "chore: release v0.2.0"`
4. 打 tag：`git tag v0.2.0`
5. 推：`git push && git push --tags`
6. 在 GitHub 用 tag 创建 Release，正文用 changelog 内容

## 代码风格

- 变量名清晰、有描述性
- 复杂逻辑加注释
- 函数小而专一
- Bash 脚本用可移植的 shebang，比如 `#!/usr/bin/env bash`，别硬编码 `/bin/bash`
- 改动要充分测试

## 有问题？

大改之前先开 issue 讨论。
