# Claudesidian: Claude Code + Obsidian 启动套件

把你的 Obsidian vault 变成 AI 驱动的第二大脑。

## 这是什么？

一套预配置好的 Obsidian vault 结构，和 Claude Code 无缝协作，让你能：

- 把 AI 当成 thinking partner，不只是写作助手
- 用 PARA 方法组织知识
- 用 Git 做版本控制
- 在任何地方访问你的 vault（包括手机）

## 快速上手

### 1. 拿到启动套件

**方案 A：用 Git Clone**

```bash
# 用你想要的文件夹名（把 'my-vault' 换成任何名字）
git clone https://github.com/heyitsnoah/claudesidian.git my-vault
cd my-vault

# 例子：
# git clone https://github.com/heyitsnoah/claudesidian.git obsidian-notes
# git clone https://github.com/heyitsnoah/claudesidian.git knowledge-base
# git clone https://github.com/heyitsnoah/claudesidian.git second-brain
```

**方案 B：下载 ZIP（不需要 Git）**

1. 在 GitHub 点 "Code" → "Download ZIP"
2. 解压到你想放的位置
3. 在 Claude Code 里打开这个目录

### 2. 跑 Setup Wizard

```bash
# 在目录里启动 Claude Code
claude

# 跑交互式 setup wizard（在 Claude Code 里）
/init-bootstrap
```

它会做这些事：

- 自动装依赖
- 断开和原始 claudesidian 仓库的连接
- **智能分析**你已有的 vault 结构和模式
- **安全导入**你已有的 Obsidian vault 到 OLD_VAULT/（如果你有的话）
- **研究你的公开作品**用于个性化上下文（需要你授权）
- 询问你的工作流偏好
- 创建个性化的 CLAUDE.md 配置
- 搭好目录结构
- 可选：配置 Gemini Vision 用于图像 / 视频分析
- 可选：配置 Firecrawl 用于网页调研
- 初始化 Git 用于版本控制

### 3. 在 Obsidian 里打开（可选但推荐）

- 下载 [Obsidian](https://obsidian.md)
- 从 claudesidian 文件夹打开 vault
- 这样你能在 Claude Code 之外有一个可视界面

### 4. 第一次会话

告诉 Claude Code：

```
我要开始一个新项目，关于 [话题]。
我现在是 thinking mode，不是 writing mode。
请搜下 vault 里相关的已有笔记，
然后通过提问帮我探索这个话题。
```

或用预配置好的 skill（在 Claude Code 里）：

```
/thinking-partner   # 协作探索
/daily-review       # 一天结束的复盘
/research-assistant # 深入研究某话题
```

## 目录结构

```
claudesidian/
├── 00_Inbox/           # 新想法的临时捕获区
├── 01_Projects/        # 活跃的限时事务
├── 02_Areas/           # 持续性责任
├── 03_Resources/       # 参考资料和知识库
├── 04_Archive/         # 完成或不活跃的事物
├── 05_Attachments/     # 图片、PDF 等
├── 06_Metadata/        # Vault 配置和模板
│   ├── Reference/      # 文档和指南
│   └── Templates/      # 可复用的笔记模板
└── .scripts/           # 自动化辅助脚本
```

## 核心概念

### Thinking Mode 和 Writing Mode

**Thinking Mode**（调研与探索）：

- Claude 提问理解你的目标
- 搜索已有笔记里的相关内容
- 帮你在想法之间建立连接
- 维护洞察和进展的日志

**Writing Mode**（内容创作）：

- 基于你的调研生成草稿
- 帮你组织和编辑内容
- 创建最终交付物

### PARA 方法

**Projects**：有截止日期和具体产出

- 例："Q4 2025 营销策略"
- 在 `01_Projects/` 里建文件夹

**Areas**：持续性、没有结束日期

- 例："健康"、"财务"、"团队管理"
- 放在 `02_Areas/`

**Resources**：长期感兴趣的话题

- 例："AI 研究"、"写作技巧"
- 存到 `03_Resources/`

**Archive**：不活跃的事物

- 完成的项目和它们的产出
- 不再相关的旧笔记

## Skills

Claudesidian 自带一些 agent skill，能在你描述任务时自动触发。Skill 在 Claude Code、OpenCode、Codex、Cursor 和 Pi 里都能用 —— 它们存在 `.agents/skills/<name>/SKILL.md`（标准位置），并通过 symlink 链接到 `.claude/skills/` 和 `.pi/skills/`。

- `thinking-partner` — 通过提问探索想法
- `inbox-processor` — 整理你的捕获条目
- `research-assistant` — 深入研究话题
- `daily-review` — 一天结束的复盘
- `weekly-synthesis` — 找出本周的模式
- `de-ai-ify` — 去掉文本里的 AI 写作痕迹
- `add-frontmatter` — 给笔记加 YAML 属性
- `download-attachment` — 把 URL 存到附件文件夹
- `pragmatic-review` — YAGNI/KISS 风格的代码审查
- `pull-request` — 从你的改动开 PR
- `release` — 发布新版 claudesidian
- `upgrade` — 升级到最新版 claudesidian
- `init-bootstrap` — 重新跑 setup wizard
- `install-claudesidian-command` — 装一个 shell 命令，从任何地方启动 vault

Skill 会在你描述需求时自动触发 —— 比如说"收个尾"会加载 `daily-review`，"开个 PR 吧"会加载 `pull-request`。不需要打 slash 命令。

### 用 `upgrade` 保持最新

Claudesidian 在你启动 Claude Code 时会自动检查更新，有新功能时提醒你跑 `upgrade` skill。

upgrade 命令会智能合并新功能，同时保护你的自定义内容：

```bash
# 预览要更新什么（推荐先看一下）
/upgrade check

# 跑交互式升级
/upgrade

# 跳过确认直接更新（高级用法）
/upgrade force
```

**升级会做什么**：

- 改之前先做带时间戳的备份
- 每个文件改之前先给你看 diff
- 保护你的个人笔记和自定义
- 只更新系统文件（命令、agent、脚本）
- 永远不动你的内容文件夹（00_Inbox、01_Projects 等）
- 必要时能回滚

**安全特性**：

- 你所有个人内容都受保护
- 完整备份建在 `.backup/upgrade-[timestamp]/`
- 文件级 diff 复查和确认
- 进度记录在 `.upgrade-checklist.md`
- 任何时候都能停下来续上

## 视觉与文档分析（可选）

配上 [Google Gemini](https://ai.google.dev/) MCP 后，Claude Code 能直接处理你的附件，不用你描述给它听。这意味着：

- **直接图像分析**：Claude 看真实的图，不是你的描述
- **PDF 文字提取**：完整文档文本，不用复制粘贴
- **批量处理**：一次分析多张截图或多个文档
- **智能整理**：基于图片内容自动生成文件名
- **对比任务**：对比改造前后的截图、设计稿等

**为什么重要**：与其让你描述"一张显示报错信息的截图"，Claude Code 直接看到并读出错误。调试 UI 问题、分析图表、处理扫描文档都很合适。

**拿 Gemini API key**：

1. 访问 [Google AI Studio](https://aistudio.google.com)
2. 用 Google 账号登录
3. 点左侧栏的 "Get API key"
4. 创建新 API key（免费！）
5. 设到环境变量：`export GEMINI_API_KEY="your-key-here"`

完整安装指南看 `.claude/mcp-servers/README.md`

## 网页调研（可选）

配上 [Firecrawl](https://www.firecrawl.dev/) 后，辅助脚本能抓完整网页内容直接存到 vault。这意味着：

- **完整文本捕获**：脚本把整篇文章文本写入文件，不是摘要
- **保留上下文**：Claude 不用把网页内容塞进 context
- **批量处理**：用 `firecrawl-batch.sh` 一次存多篇文章
- **干净的 markdown**：网页转成可读、可搜索的 markdown
- **永久归档**：你的研究永远留在 vault 里

**为什么重要**：与其让 Claude 读网页然后摘要（丢失细节），脚本会保存完整文本。然后 Claude 能搜索和分析成千上万篇存好的文章，不用担心 context 溢出。研究项目、文档归档、构建知识库都很合适。

**例子工作流**：

```bash
# 存单篇文章
npm run firecrawl:scrape -- "https://example.com/article" "03_Resources/Articles"

# 批量存多个 URL
npm run firecrawl:batch -- urls.txt "03_Resources/Research"
```

**拿 Firecrawl API key**：

1. 访问 [Firecrawl](https://www.firecrawl.dev) 注册
2. 注册送 300 免费 credits（开源，也能自部署）
3. 在 dashboard 找你的 API key
4. 复制 key（格式：`fc-xxxxx...`）
5. 设到环境变量：`export FIRECRAWL_API_KEY="fc-your-key-here"`

## 辅助脚本

用 `pnpm` 跑：

- `attachments:list` — 显示未处理附件
- `attachments:organized` — 统计已整理的文件
- `attachments:sizes` — 找大文件
- `attachments:orphans` — 找没人引用的附件
- `vault:stats` — 显示 vault 统计

## 进阶设置

### 从任何地方快速启动

装一个 shell 命令，能从任何目录启动你的 vault：

```bash
# 在 Claude Code 里跑：
/install-claudesidian-command
```

这会创建一个 `claudesidian` alias：

- 自动切到你的 vault 目录
- 尝试恢复已有会话（如果有）
- 没有则启动新会话
- 完事后回到原目录

**用法**：

```bash
# 从终端任何位置：
claudesidian

# 它会自动恢复上次会话，或启动新会话
```

命令会加到你的 shell 配置（~/.zshrc、~/.bashrc 等），跨终端会话保留。

### Git 集成

初始化 Git 做版本控制：

```bash
git init
git add .
git commit -m "Initial vault setup"
git remote add origin your-repo-url
git push -u origin main
```

最佳实践：

- 每次工作会话后 commit
- 写描述性的 commit 信息
- 开始工作前先 pull

### 移动端访问

1. 搭一个小 server（mini PC、云 VPS 或家用 server）
2. 装 Tailscale 做安全 VPN 接入
3. 把 vault clone 到 server
4. 手机上用 Termius 或类似 SSH 客户端
5. 远程跑 Claude Code

### 自定义命令

把指令存到 `.claude/commands/` 创建专门命令：

**研究助手**（`06_Metadata/Agents/research-assistant.md`）：

```markdown
You are a research assistant.

- Search the vault for relevant information
- Synthesize findings from multiple sources
- Identify gaps in knowledge
- Suggest areas for further exploration
```

## 几条经验之谈

### 实践得来的

1. **先进 thinking mode**：抑制立刻生成内容的冲动
2. **当 token maximalist**：上下文越多结果越好
3. **什么都存**：聊天、片段、不完整的想法都留下来
4. **信任但要验证**：AI 生成的内容一定要读
5. **大胆中断**：AI 帮你轻松续上

## 排错

### Claude Code 找不到我的笔记

- 确认从 vault 根目录启动 Claude Code
- 检查文件权限
- 确认 markdown 文件后缀是 `.md`

### Git 冲突

- 开始工作前一定 pull
- 频繁 commit，写清楚的信息
- 实验性改动用分支

### 附件管理

- 跑 `npm run attachments:create-organized` 建好目录
- 用辅助脚本找孤儿文件
- 附件控制在 10MB 以下（Git 友好）

## 哲学

这套系统基于几条核心原则：

1. **AI 放大思考，不只是写作**
2. **本地文件 = 完全控制**
3. **结构使创造成为可能**
4. **迭代胜过完美**
5. **目标是洞察，不只是信息**

## 贡献

欢迎社区贡献！这是一个活的模板，靠每个人的输入变得更好。

### 怎么贡献

1. **Fork 仓库**
2. **建 feature 分支**（`git checkout -b feature/amazing-feature`）
3. **改你想改的**
4. **测一下**确保没坏
5. **Commit**（`git commit -m 'Add amazing feature'`）
6. **推到分支**（`git push origin feature/amazing-feature`）
7. **开 PR**，写清楚改了什么

### 我们想要的

- **新命令**：常见工作流的实用 Claude Code 命令
- **新 agent**：针对特定任务的专门 agent
- **文档改进**：更好的解释、例子、指南
- **Bug 修复**：发现坏东西？修了它！
- **工作流模板**：分享你高效的工作流
- **辅助脚本**：让 vault 管理更轻松的自动化工具
- **集成指南**：把 Claudesidian 和其他工具连起来
- **核心更新**：升级系统、setup wizard 或其他核心功能的改进

### 准则

- 命令保持单一目的
- 写清楚的文档，带例子
- 提交前充分测试
- 遵循已有代码风格和结构
- 在 CHANGELOG.md 加你的改动
- **AI 生成的内容欢迎，但提交前你必须仔细读和审查每一行** — 永远别提交你自己看不懂的代码

### 拿到更新

新功能合并后，用户能轻松拿到：

```bash
/upgrade
```

upgrade 命令智能合并新功能，同时保护你的个人定制，让你能享受到社区贡献而不丢工作。

### 有问题或想法？

- 大改之前开 issue 讨论
- 加入已有 issue 的讨论
- 分享你的用例 — 帮我们更好理解需求

记住：最佳实践来自使用，不是理论。你真实的经验让这套系统对所有人都更好！

## 资源

- [Obsidian 文档](https://help.obsidian.md)
- [PARA Method](https://fortelabs.com/blog/para/)
- [Claude Code 文档](https://claude.ai/docs)

## 灵感来源

这个启动套件的灵感来自：

- [How to Use Claude Code as a Second Brain](https://every.to/podcast/how-to-use-claude-code-as-a-thinking-partner) —
  Noah Brier 和 Dan Shipper 的访谈
- 由 [Alephic](https://alephic.com) 团队搭建 — 一家 AI-first 的战略与软件合作伙伴，帮助组织通过定制 AI 系统解决复杂问题

## 许可证

MIT — 想怎么用就怎么用，让它变成你自己的。

---

_记住：自行车一开始骑起来摇摇晃晃，然后某天你会忘记它曾经难过。_
