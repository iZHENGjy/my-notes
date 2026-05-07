# Changelog

claudesidian 所有重要变更都记录在这。

格式参考 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，
版本号遵循 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)。

## [Unreleased]

## [0.15.1] - 2026-04-11

### Fixed

- 项目脚本里的 Bash shebang 改成可移植形式（`#!/usr/bin/env bash`），不再硬编码 `/bin/bash`。修复了 NixOS 等系统上 `/bin/bash` 不是标准路径时脚本跑不起来的问题。感谢 @jamestrew (#37)。

## [0.15.0] - 2026-04-10

### Added

- 多 agent skill 布局：skill 现在存在 `.agents/skills/<name>/SKILL.md`（标准位置，OpenCode、Codex、Cursor 原生读取），并通过 symlink 链到 `.claude/skills/<name>` 和 `.pi/skills/<name>`，这样 Claude Code 和 Pi 也能找到。改标准位置的，五个 agent 全跟着变。
- `pnpm skills:validate` 脚本 — 跑打包好的 `quick_validate.py` 校验每个 skill，报告 frontmatter 格式问题，遇到第一个失败就非零退出。
- 打包了 Anthropic 官方的 `skill-creator` skill（来自 [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/skill-creator)），包括完整的 agents/、scripts/、references/、assets/ 目录树，连同它的 `LICENSE.txt`（Apache 2.0）和指向上游第三方声明的 `NOTICE.md`。

### Changed

- 把 `.claude/commands/` 里所有 14 个 slash 命令转成了 agent skill，加上正确的 `name` + `description` frontmatter。正文原样保留，只修了一些过时的引用。
- 重写了 `upgrade` skill（642 → 276 行），适配新布局。增加了布局检测（跳过老的 `.claude/commands/` vault）、用 `-L` 和 `readlink` 安全修复错误 / 损坏 symlink 的 hygiene 步骤、还有给老用户的迁移路径文档。所有原始安全机制都保留（备份、强制 diff、不自动选、绝不 `cp -f`、"绝不动的文件"清单、回滚说明）。
- 重写了 `init-bootstrap` skill 的 "available commands" 段，指向用自然语言触发的 skill。把失效的 `/setup-gemini` 和 `/setup-firecrawl` 引用换成具体的内联设置说明。
- 重写了 README 的 "Claude Code Commands" 段，改成介绍 skill，去掉失效的 `/[command-name]` 调用提示和失效的 `create-command` 条目。

### Fixed

- `install-claudesidian-command` skill：代码审查发现的两个老的数据丢失 bug。
  1. `sed` 范围删除 `/alias|function/,/^end$/d` 会从一个 bash alias（没有 `end`）开始一路删到文件里下一个 `^end$` 关键字，可能炸掉无关的 shell 配置。拆成单行删 alias、范围删 fish function。
  2. 路径转义顺序反了（先转引号再转反斜杠，导致新加的反斜杠被双重转义）。改成先转反斜杠。

### Removed

- `.claude/claude_config.json` — 没有任何代码引用的死配置文件。
- `.claude/commands/README.md` — 一个空目录的过时文档。
- `.claude/commands/create-command.md` — 不再需要。

### Migration

- 还在 `.claude/commands/` 布局上的 vault：跑 `upgrade` skill。它会检测到老布局并一步步带你迁移（把命令文件挪到 `.agents/skills/<name>/SKILL.md`、用 `add-frontmatter` 加 frontmatter、建 symlink、校验）。迁移是一次性的，成功后未来升级走正常流程。
- 关闭 #16（升级到使用 Skill）、#17（unknown slash command: /setup-gemini and /setup-firecrawl）、#30（OpenCode 兼容性）。

## [0.14.2] - 2026-01-13

### Changed

- `@modelcontextprotocol/sdk` 升级到 v1.25.2（修复了 HIGH 级别的 ReDoS 和 DNS rebinding 漏洞）
- 升级 dev 依赖（eslint、prettier、typescript-eslint）到最新版本

### Added

- 给 kepano/obsidian-skills 加了 MIT 许可证归属（json-canvas、obsidian-bases、obsidian-markdown）
- 把 `.worktrees/` 加到 gitignore，支持 git worktree 工作流

### Security

- 修复 5 个 Dependabot 安全告警（3 high、2 moderate）

## [0.14.1] - 2026-01-13

### Fixed

- `firecrawl-batch.sh` 文件名清洗时保留 CJK（中日韩）字符
- `/upgrade` 命令里把 sed 换成 cut，兼容 zsh

## [0.14.0] - 2026-01-13

### Added

- 新的 `.claude/skills/` 目录，自动触发的 skill 系统
  - `obsidian-markdown` — Obsidian Flavored Markdown 参考（wikilink、callout、embed）
  - `obsidian-bases` — .base 文件创建和编辑指南
  - `json-canvas` — .canvas 文件创建和编辑指南
  - `systematic-debugging` — 4 阶段调试方法论
  - `skill-creator` — 创建新 skill 和命令的指南
  - `git-worktrees` — Git worktree 工作流指南
- 新增 `/pragmatic-review` 命令，做 YAGNI/KISS 风格的代码审查
  - 默认模式：快速扫描过度工程
  - Deep 模式（--deep）：6 趟全面审查
  - CI 模式（--ci）：非交互式，用于 GitHub Actions
- Skill 发现 hook：用户提到 "skill" 时列出可用 skill
- `.prettierignore` 加入 markdown 文件排除

## [0.13.1] - 2025-10-13

### Fixed

- 修正硬编码的目录路径，把空格换成下划线
  - 修 `transcript-extract.sh`：`00 Inbox/Clippings` → `00_Inbox/Clippings/`
  - 修 `firecrawl-batch.sh`：`00 Inbox/Clippings` → `00_Inbox/Clippings/`
  - 修 `update-attachment-links.js`：`05 Attachments` → `05_Attachments`
  - 修 `fix-renamed-links.js`：`05 Attachments` → `05_Attachments`
  - 修 `GEMINI_VISION_QUICK_START.md` 里的示例路径
- 移除 `firecrawl-batch.sh` 里的安全风险，去掉 `source ~/.zshrc`
- 修正 Gemini Vision 文档里指向不存在文件的引用

### Added

- `firecrawl-batch.sh` 加自定义输出目录的 flag（`-o|--output-dir`）
- README.md 里增加新输出目录选项的文档

## [0.13.0] - 2025-01-17

### Added

- 新的 `/install-claudesidian-command`，创建 shell alias / function 从任何地方启动 vault
- `/init-bootstrap` 支持检测和导入 iCloud Drive vault（仅 macOS）
- 启动器命令支持 Fish shell，用正确的 function 语法
- 全面的用户输入路径校验，带友好的错误提示
- 平台检测，跨平台兼容（Linux、macOS、Windows）

### Fixed

- 启动器命令里关键的 shell 注入漏洞，改成正确的路径转义
- 修改 shell 配置前先备份
- iCloud 同步状态检查，下载没完成时给出软警告
- Shell 检测改用默认 shell，不再用当前会话 shell
- 已有 alias / function 的替换需要用户确认
- 改进错误处理文档，加入解释

### Security

- 带空格、引号、特殊字符的 vault 路径正确转义
- 修改 shell 配置前做带时间戳的备份
- 用户提供的 vault 路径加输入校验
- 文档里加安全考量段

## [0.12.1] - 2025-01-17

### Fixed

- 改进 upgrade 命令，避免交互式提示和卡住
- 增强 upgrade 命令，加显式备份步骤和非交互式文件替换
- 基于实际使用反馈打磨 upgrade 命令，更可靠
- 给 upgrade 命令应用 lint 修复，提升代码质量

## [0.12.0] - 2025-01-17

### Added

- 新的 `/download-attachment` 命令，把网页内容和文件下载到 Obsidian 附件目录
- 新的 `/pull-request` 命令，做智能变更分析并生成 PR 描述
- 两个命令都和 Obsidian vault 结构无缝集成

### Fixed

- 基于 PR 评审反馈，改进下载命令的安全性
- 调整代码格式，通过 lint

## [0.11.0] - 2025-01-17

### Added

- 全面的 lint 和 formatter 配置（ESLint + Prettier）
- 配置文件统一放到 `.config/` 文件夹，结构更清晰
- GitHub Action 工作流，PR 自动跑 lint 检查
- 指定 package manager，依赖管理一致

### Fixed

- 解决 lonely if ESLint 警告，代码更干净
- 修复 GitHub Action 的 pnpm 配置
- 加 packageManager 字段，确保不同环境工具一致

## [0.10.1] - 2025-01-15

### Fixed

- 修正 GitHub Action 配置，用 claude_args 替代 allowed_tools
- 修复 allowed tools 参数的工作流校验错误

## [0.10.0] - 2025-01-15

### Added

- Claude Code 集成的 GitHub Actions 工作流
- Claude 现在能响应 issue 和 PR 里的 @claude 提及
- 配置 Claude 创建 PR 和推送变更的权限
- 加入 pnpm、git、gh CLI 操作的 allowed tools 配置
- 出于安全考虑，限制只有仓库 owner 能用
- init-bootstrap 加开发工作流选项
  - 个人 vault 用户：删除 .github 文件夹
  - 贡献者：保留 GitHub 工作流用于开发

## [0.9.2] - 2025-01-14

### Fixed

- 修正 Firecrawl 脚本示例，改用 `npm run` 命令
- 加贡献准则：提交前要仔细审查 AI 生成的内容

### Removed

- 移除 README 里的 Common Patterns 段（重复）

## [0.9.1] - 2025-01-14

### Changed

- 增强 Gemini Vision 文档，解释直接处理图片 / PDF 的好处
- 增强 Firecrawl 文档，解释完整文本捕获和上下文保留
- 加 Gemini 和 Firecrawl 的详细 API key 设置说明

### Removed

- 移除 README 里的 Essential Workflows 段（和命令描述重复）

## [0.9.0] - 2025-01-14

### Added

- 增强 upgrade 命令文档，加详细用法示例和安全特性
- 加贡献相关段，社区贡献指南
- 加 MIT 许可证文件，开源许可清晰

### Changed

- 改进 Claude Code 命令 vs agent 区别的文档说明
- 更新贡献指南，鼓励命令、agent 和核心更新方面的 PR

### Removed

- 移除 thinking-partner agent（只保留 slash 命令）

## [0.8.8] - 2025-01-13

### Fixed

- 在 upgrade 命令文档里加关键警告
- 强调改动前必须显示 diff 的要求
- 加正确 vs 错误实现的例子
- 防止升级时丢失用户自定义

## [0.8.7] - 2025-01-13

### Fixed

- Release 命令现在会用 gh CLI 自动创建 GitHub release
- 防止漏建 GitHub release（v0.8.2 到 v0.8.5 就漏过）
- 从 CHANGELOG.md 抽取 release notes 作为 GitHub release 正文

## [0.8.6] - 2025-01-13

### Changed

- 改进 release 命令里的语义化版本指南，决策指引更清晰

## [0.8.5] - 2025-01-13

### Fixed

- 升级命令现在能在没 git 连接时也工作（适合断开连接的用户）
- 把最新版 clone 到 .tmp/ 目录，不再要求上游 remote
- 用 .tmp/ 而不是 /tmp/，对 Obsidian 隐藏升级文件
- 检测到本地修改时加用户选择提示
- 加校验步骤，确保所有文件都被处理
- `.tmp/` 加到 .gitignore，仓库更干净

## [0.8.4] - 2025-01-13

### Fixed

- 简化升级命令，系统化检查所有系统文件
- 创建升级 checklist 跟踪逐文件进度
- 升级只针对 claudesidian 系统文件，不动用户内容
- 加显式的逐文件 diff 复查（更新前）
- 升级流程支持暂停 / 续上（用 checklist 跟踪）

## [0.8.3] - 2025-01-13

### Fixed

- 改进 init-bootstrap 在多 vault 情况下的选择
- 导入任何 vault 前都加显式确认
- 增强用户身份识别提示，加更好的解释
- 调研用户时加歧义消除提示，找对人
- 明确指令：没有显式 vault 确认绝不进行

## [0.8.2] - 2025-01-13

### Fixed

- 改进 SessionStart hook 的格式化，命令前加箭头标记
- 修复更新通知显示，输出干净文本而不是原始 JSON
- 增强首次运行和更新消息的视觉布局

## [0.8.1] - 2025-01-13

### Changed

- 更新 README 加全面的功能描述：
  - 智能 vault 分析和模式检测
  - 用户调研和画像构建
  - 自动更新通知系统
  - Firecrawl 网页调研能力
  - 完整可用命令列表（包括 /upgrade）

## [0.8.0] - 2025-01-13

### Added

- 启动会话时自动检查更新
- SessionStart hook 从 GitHub 取最新版本和本地比较
- 有新版时弹更新通知
- check-updates npm 脚本做版本比较
- 即使断开和原始仓库的连接也能工作

### Changed

- 增强 release 命令文档，语义化版本指南更清晰
- 在 commit 里何时用 feat: vs fix: vs refactor: 给了更好指引

## [0.7.0] - 2025-01-13

### Added

- 全面的 vault 分析（用 tree、笔记采样、模式检测）
- 增强画像构建（URL 抓取 + 自定义 context）
- CLAUDE.md 里的时间戳支持动态日期生成
- Gemini Vision 和 Firecrawl 设置加 "Later" 选项
- 多种 vault 导入选项（yes/no/skip/path）
- 更深入的调研能力，带歧义消除确认

### Changed

- init-bootstrap 现在先分析 vault 结构再导入
- 单个搜索结果也总是要确认用户身份
- 等用户选完组织方式之后再创建文件夹
- 自动检测插件和附件，不再问用户
- Firecrawl 包装成调研利器，解释更好
- 画像构建从提供的 URL 抓全面背景

### Fixed

- 文件计数不再带深度限制
- 导入和个性化问题的顺序正确
- 从已有 vault 更精准检测用户偏好

## [0.6.0] - 2025-01-13

### Added

- 智能 vault 导入：保留已有结构到 OLD_VAULT 文件夹
- 自动检测已有 Obsidian vault（搜 .obsidian 文件夹）
- 用户调研和歧义消除，做个性化设置
- 自动检测文件命名模式和文件夹组织
- vault 配置存到 .claude/vault-config.json 备查
- 设置每一步加教学性解释
- 同时支持 pnpm 和 npm 包管理器
- 全面的 Obsidian 文件复制（插件、回收站、设置）
- clone 仓库时给文件夹命名的清晰例子

### Changed

- init-bootstrap 现在能安全导入整个 vault 结构而不丢数据
- Gemini Vision 和 Firecrawl 提示明确说明工具已包含
- README 加 clone 时自定义文件夹名的例子
- 自动检测模式，去掉不必要的问题

## [0.5.0] - 2025-01-13

### Added

- 用 SessionStart hook 显示首次运行欢迎消息
- 用 FIRST_RUN 标记文件检测全新安装
- Markdown 格式的欢迎提示，带设置说明
- 新用户的自动检测和指引

### Changed

- init-bootstrap 完成设置后会移除 FIRST_RUN 标记
- Hook 配置用内联命令（不需要外部脚本）

## [0.4.0] - 2025-01-13

### Added

- 简化设置流程，增强 init-bootstrap 命令：
  - 自动断开和原仓库的连接
  - 给非 git 用户提供文件夹重命名辅助
  - 同时支持 git clone 和 ZIP 下载方式
  - 导入已有 Obsidian vault 的清晰提示
- README 加多种设置路径，照顾不同水平用户

### Changed

- init-bootstrap 现在处理完整的环境设置（包括 git 管理）
- README 更新，给技术 / 非技术用户都有更清晰的快速上手

## [0.3.1] - 2025-01-13

### Fixed

- 移除仓库里的 CLAUDE.md 和 settings.local.json
- 这些用户专属文件改由 init-bootstrap 本地生成
- 两个文件加到 .gitignore 防止误提交
- 让新用户拿到的仓库结构更干净

## [0.3.0] - 2025-01-13

### Added

- 智能升级命令（`/upgrade`），用 AI 做语义合并
- 智能冲突解决：保留用户自定义，同时加新功能
- 自动备份系统，支持回滚
- 选择性更新分类（AI 可合并、自动安全、受保护）
- 基于 2025 年 LLM 驱动代码迁移的最佳实践

## [0.2.3] - 2025-01-13

### Changed

- 更新 init-bootstrap 命令和设置配置

## [0.2.2] - 2025-01-13

### Fixed

- 修正所有文档使用正确的 slash 命令语法（/command-name）
- 修复显示错误 'claude run' 语法的例子
- 更新 README、CLAUDE.md、install.sh 和命令文档

## [0.2.1] - 2025-01-13

### Changed

- 更新 README，用 init-bootstrap 命令替代 install.sh
- 简化快速上手为 2 步流程
- README 里加预配置命令的例子

## [0.2.0] - 2025-01-13

### Added

- Release 命令做自动版本管理和发布
- Gemini Vision 视频分析支持：
  - 本地视频文件（MP4、AVI、MOV、WebM、MKV、WMV、FLV、3GP、M4V）
  - 直接分析 YouTube URL，不用下载
  - 自动检测视频处理状态
- 文档加视频分析示例

### Changed

- 增强 init-bootstrap 命令：完整的环境设置（含 MCP 配置）
- 更新 Gemini Vision MCP server 支持视频格式

## [0.1.0] - 2025-01-13

### Added

- claudesidian 首发版本 — Claude Code + Obsidian 启动套件
- PARA 方法的目录结构（00_Inbox 到 06_Metadata）
- 通过 `claude run init-bootstrap` 的 bootstrap 初始化系统
- 预配置的 Claude Code 命令：
  - thinking-partner — 协作思考模式
  - inbox-processor — 整理捕获条目
  - research-assistant — 深入研究话题
  - daily-review — 一天结束的复盘
  - weekly-synthesis — 找出本周的模式
  - create-command — 创建新自定义命令
  - de-ai-ify — 去除 AI 写作痕迹
  - add-frontmatter — 给笔记加元数据
  - init-bootstrap — 交互式 setup wizard
- 专门工作流的 Claude Code agents
- 辅助脚本：
  - firecrawl-batch.sh — 批量网页抓取
  - firecrawl-scrape.sh — 单 URL 抓取
  - fix-renamed-links.js — 重命名后修复坏链接
  - update-attachment-links.js — 更新附件引用
  - transcript-extract.sh — 提取 YouTube 字幕
  - vault-stats.sh — 显示 vault 统计
- 通过 pnpm 提供的附件管理命令
- Gemini Vision MCP server 用于图像 / PDF 分析（可选）
- CLAUDE-BOOTSTRAP.md 配置模板
- 全面的 README 设置说明
- 自动化设置的 install 脚本
- Git 集成（含正确的 .gitignore）

### Changed

- 用动态的 init-bootstrap 命令替代静态 CLAUDE.md

### Security

- API key 存到环境变量
- `.mcp.json` 加到 gitignore 防泄露

[Unreleased]: https://github.com/heyitsnoah/claudesidian/compare/v0.15.1...HEAD
[0.15.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.15.0...v0.15.1
[0.15.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.14.2...v0.15.0
[0.14.2]: https://github.com/heyitsnoah/claudesidian/compare/v0.14.1...v0.14.2
[0.14.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.14.0...v0.14.1
[0.14.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.13.1...v0.14.0
[0.13.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.13.0...v0.13.1
[0.13.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.12.1...v0.13.0
[0.12.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.12.0...v0.12.1
[0.12.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.11.0...v0.12.0
[0.11.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.10.1...v0.11.0
[0.10.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.10.0...v0.10.1
[0.10.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.9.2...v0.10.0
[0.9.2]: https://github.com/heyitsnoah/claudesidian/compare/v0.9.1...v0.9.2
[0.9.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.9.0...v0.9.1
[0.9.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.8...v0.9.0
[0.8.8]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.7...v0.8.8
[0.8.7]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.6...v0.8.7
[0.8.6]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.5...v0.8.6
[0.8.5]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.4...v0.8.5
[0.8.4]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.3...v0.8.4
[0.8.3]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.2...v0.8.3
[0.8.2]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.1...v0.8.2
[0.8.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.8.0...v0.8.1
[0.8.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.7.0...v0.8.0
[0.7.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.6.0...v0.7.0
[0.6.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.5.0...v0.6.0
[0.5.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.4.0...v0.5.0
[0.4.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.3.1...v0.4.0
[0.3.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.3.0...v0.3.1
[0.3.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.2.3...v0.3.0
[0.2.3]: https://github.com/heyitsnoah/claudesidian/compare/v0.2.2...v0.2.3
[0.2.2]: https://github.com/heyitsnoah/claudesidian/compare/v0.2.1...v0.2.2
[0.2.1]: https://github.com/heyitsnoah/claudesidian/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/heyitsnoah/claudesidian/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/heyitsnoah/claudesidian/releases/tag/v0.1.0
