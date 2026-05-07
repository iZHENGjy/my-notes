# ⚙️ Metadata

Vault 的配置、文档、组织工具放这里。

## 干啥用的

metadata 文件夹放：
- 关于 vault 本身的文档
- 创建笔记用的模板（保持一致）
- 参考指南、how-to
- agent 配置
- 工作流文档

## 结构

```
06_Metadata/
├── Reference/         # 指南和文档
├── Templates/        # 笔记模板
├── Agents/          # Claude Code agent 配置
├── Workflows/       # 流程文档
└── Archive/        # 旧配置
```

## 各目录放什么

### Reference
- 本 vault 的文档
- Claude Code prompt 库
- 风格指南
- 工作流文档
- 学习资料

### Templates
- 项目模板
- 每日笔记模板
- 会议模板
- 研究模板
- 复盘模板

### Agents
- Thinking partner 指令
- 研究助手配置
- 编辑 agent 配置
- 自定义 agent 定义

### Workflows
- 周复盘流程
- 项目完成 checklist
- Inbox 处理指南
- 归档流程

## 用模板

### 手动
1. 复制模板内容
2. 新建笔记
3. 粘贴 + 填空

### 用 Claude Code
```
按 project template 建一个新项目，
名字叫 [项目名]，放到 01_Projects 里。
```

## 创建自定义 Agent

把 agent 指令存成 markdown 文件：

```markdown
# Agent: [Name]

You are a [role description].

## Core Behaviors
- Behavior 1
- Behavior 2

## Workflow
1. Step 1
2. Step 2

## Constraints
- Don't do X
- Always do Y
```

然后在 Claude Code 里引用：
```
用 06_Metadata/Agents/[agent].md 里的指令，
帮我做 [task]。
```

## Claude Code 提示词

### 模板使用
```
看下 06_Metadata/Templates 里有什么模板。
用合适的模板新建一个 [type] 笔记。
```

### 文档
```
查下 06_Metadata/Reference 里关于 [话题] 的文档。
基于刚学到的更新一下指南。
```

### 跑工作流
```
跑 06_Metadata/Workflows 里的周复盘流程。
带我一步步走。
```

## 维护

### 定期更新
- 基于实际使用更新模板
- 新冒出来的工作流要写文档
- 过时的配置归档
- 参考文档保持最新

### 版本控制
- 跟踪工作流变更
- 记录为啥改
- 留旧版本归档
- 大改时加日期

## 几条建议

- **边做边记** — 工作流要趁热写下来
- **模板要迭代** — 基于使用改进
- **分享配置** — 对你管用的可能也帮到别人
- **保持简单** — 复杂系统会崩
- **加日期** — 上下文很重要

## 记住

Metadata 是你 vault 的操作系统。好的 metadata 意味着结构一致、流程可重复、能扩展生长。这里不仅记录"你知道什么"，更记录"你怎么干活"。
