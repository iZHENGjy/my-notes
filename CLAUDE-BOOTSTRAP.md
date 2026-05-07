# Obsidian Vault 指南 — Bootstrap 模板

**Claude Code + Obsidian 上手指南**

## 快速设置

1. **每次开工**：先 `git pull` 同步最新变更
2. **改完之后**：commit 并 push，保存你的工作
3. **优先用内置工具**：网页内容用 WebSearch 和 WebFetch

## 版本控制最佳实践

**关键 — 每次开工都做**：每个 Claude 新会话开始时跑 `git pull`，确保你拿到了远程仓库的最新变更。

**Commit 工作流**：

- 新建笔记后：`git add .` → `git commit -m "message"` → `git push`
- 重大编辑后：立刻 commit 和 push
- 用 `git status` 检查有没有改动
- agent 改了文件：一定要 commit 那些变更

## 目录结构（PARA 方法）

```
vault/
├── 00_Inbox/           # 临时捕获区
├── 01_Projects/        # 限时事务
├── 02_Areas/           # 长期责任
├── 03_Resources/       # 参考资料
├── 04_Archive/         # 完成 / 不活跃的
├── 05_Attachments/     # 图片、PDF 等
│   └── Organized/      # 已处理的附件
└── 06_Metadata/        # 文档和模板
    ├── Reference/      # 指南和标准
    ├── Plans/          # 战略文档
    └── Templates/      # 可复用结构
```

## PARA 方法详解

### Projects（01）

- 限时事务，有明确的完成标准
- 例：写一篇论文、做一份演讲
- 推荐子目录：Research/、Drafts/、References/、Output/

### Areas（02）

- 持续性责任，没有结束日期
- 例：健康、财务、职业成长
- 建专门的笔记，链接到相关资源

### Resources（03）

- 感兴趣的话题，留作参考
- 按主题组织的知识库
- 用于不绑定具体项目的信息

### Archive（04）

- 完成或不活跃的事物
- 维持和活跃区一样的目录结构
- 定期回看是否要重启

## Inbox 管理

### 核心原则

- Inbox 是临时的，不是永久存储
- 每周用「捕获 → 处理 → 组织」的流程处理
- 任何时候保持条目数 < 20

### 留在 Inbox 的文件

- **关键**：带数字前缀（00-06）的文件永远留着
- 最近的日 / 周总结（近 3 个月）
- 正在处理的活跃笔记

### 处理流程

1. 删过时信息
2. 把相关材料挪到 PARA 对应位置
3. 把行动转成项目任务
4. 还需要继续处理的打 `#needs-processing` 标签

## 文件组织规范

### 命名规范

- 每日笔记：`YYYY-MM-DD - Topic`
- 会议笔记：`Meeting - [Topic] - YYYY-MM-DD`
- 想法：`Idea - [简短描述]`
- 资源：`Resource - [话题] - [来源]`

### 移动规则

- 用 `mv` 命令（不用 `cp`），避免重复
- 先确认目标目录存在
- 移动后更新内部链接
- 整理时加 YAML frontmatter

## 附件管理

### 组织

- 所有非文本文件存到 `05_Attachments/`
- 处理过的文件 → `05_Attachments/Organized/`
- 命名：`[相关笔记]_[描述].[扩展名]`

### 辅助脚本

```bash
pnpm attachments:list        # 列出未处理文件
pnpm attachments:organized   # 统计已整理的文件
pnpm attachments:orphans     # 找无引用的文件
pnpm attachments:update-links # 移动后更新链接
```

## 网页内容工作流

### 内置工具（优先）

- **WebSearch**：通用网页搜索
- **WebFetch**：抓特定 URL
- 按内容类型存到合适的目录

### 自定义脚本（必要时）

- 单 URL：`pnpm firecrawl:scrape <url> <output>`
- 批量 URL：`pnpm firecrawl:batch <url1> <url2>`
- 默认存到 `00_Inbox/Clippings/`，带 frontmatter

## 写作风格指南

### 结构

- 内部引用用 `[[WikiLinks]]`
- 加 YAML frontmatter（日期、标签、状态）
- Markdown 格式一致
- 标签具体且一致

### 风格偏好

- 直接、自信的陈述
- 别用陈词滥调的过渡语
- 让陈述自己站住脚
- 不要不必要的引子

## 给 AI 助手的指南

### 整理前

1. 完整画出目录结构：`find . -type d | sort`
2. 写到 `06_Metadata/STRUCTURE.md`
3. 确认所有目标目录存在

### 处理内容时

- 尊重数字编号的核心文件（绝不挪 00-06 前缀的文件）
- 整理时一律用 `mv`，不用 `cp`
- 保护并更新双向链接
- 加合适的 YAML frontmatter

### 命令保持简单

- **必须**：直接、基础的命令，不带过滤
- **禁止**：复杂正则、管道命令、带过滤器的 find
- 对的例子：`ls -1` 然后手选文件
- 错的例子：`ls | grep pattern` 或 `find . -name "*.png"`

## 日常工作流

### 一天开始

1. 跑 `git pull`
2. 查 inbox 有什么要处理
3. 看活跃项目

### 一天结束

1. 处理新的 inbox 条目
2. Commit 并 push
3. 更新项目笔记

### 周复盘

1. 处理整个 inbox
2. 归档已完成项目
3. 更新 area 笔记
4. 复盘和整合资源

## 项目生命周期

### 启动项目

1. 在 `01_Projects/[ProjectName]` 建文件夹
2. 加子目录：Research/、Drafts/、Output/
3. 建 README，写目标和时间线

### 项目进行中

- 所有相关材料留在项目文件夹
- 链接到相关资源和 area
- 定期 commit 跟踪进度

### 项目完成

1. 写项目总结
2. 整个文件夹挪到 `04_Archive/`
3. 更新相关 area 笔记
4. Commit 时写完成信息

## 最佳实践

### 组织

- 目录层级保持浅（最多 3 层）
- 7 个以上相关笔记才建子目录
- 多用链接，少深嵌套
- 主要目录加 README

### 内容创作

- 先捕获，再整理
- 一笔记一想法
- 大胆加链接
- 标签一致

### 维护

- 每周处理 inbox
- 每月复盘项目
- 每季度清理归档
- 定期 git commit

---

_这是 bootstrap 模板。基于你的工作流和需要做定制。_
