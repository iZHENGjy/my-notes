---
name: add-frontmatter
description: Add or update YAML frontmatter properties to enhance Obsidian note organization. Use when the user asks to add, fix, normalize, or improve frontmatter, properties, metadata, tags, or YAML on a note or folder of notes. / 给 Obsidian 笔记加 / 更新 YAML frontmatter，增强组织性。当用户要"加 frontmatter"、"修 properties"、"规范化 metadata / tags / YAML"（针对单个笔记或一个文件夹）时触发。
---

你的任务是分析 Obsidian 笔记，加上智能的 YAML frontmatter 属性，增强组织性和可发现性。

## 输入

- 路径：要处理的文件或文件夹
- 当前日期：用系统日期

## 任务

### Step 1：识别要处理的笔记

```bash
# 单个文件
读取指定文件

# 文件夹
找出文件夹里所有 .md 文件
```

### Step 2：分析笔记内容

每个笔记看：

- 主要话题和主题
- 笔记类型（meeting、daily、reference、project）
- 关键实体（人、项目、日期）
- 已有属性（保留有效的）
- 标题质量（必要时改 / 加）

### Step 3：生成合适的属性

#### 各类型笔记的标准属性

**会议笔记**：

```yaml
---
title: [描述性的会议标题]
date: YYYY-MM-DD
type: meeting
attendees: ['Person 1', 'Person 2']
project: Project Name
tags: [meeting, project-name]
action_items:
  - 'Action item 1'
  - 'Action item 2'
status: complete
---
```

**每日笔记**：

```yaml
---
title: Daily Note - YYYY-MM-DD
date: YYYY-MM-DD
type: daily-note
tags: [daily]
highlights:
  - 'Key event or thought'
mood: productive
---
```

**参考 / 文章笔记**：

```yaml
---
title: [文章或概念标题]
type: reference
source: "[[Source Note]]" or URL
author: Author Name
date_saved: YYYY-MM-DD
tags: [topic1, topic2]
key_concepts: [concept1, concept2]
---
```

**项目笔记**：

```yaml
---
title: [项目名 - 组件]
type: project
status: in-progress
deadline: YYYY-MM-DD
stakeholders: ['Person 1', 'Team 2']
tags: [project, area]
priority: high
---
```

### Step 4：应用属性

每个笔记：

1. 检查是否已有 frontmatter
2. 合并新属性（不重复）
3. 修复废弃格式：
   - `tag` → `tags`
   - `alias` → `aliases`
   - `cssclass` → `cssclasses`
4. 保证 YAML 语法有效

### Step 5：更新文件

```yaml
# 格式：
---
property: value
list_property: ['item1', 'item2']
date_property: YYYY-MM-DD
linked_property: '[[Note Name]]'
---
[原内容]
```

## 属性指南

### 命名规范

- 用小写 + 下划线：`date_created`、`action_items`
- 和已有 vault 模式保持一致
- 清晰胜过聪明

### 值类型

- **文本**：简单字符串，链接要加引号
- **列表**：多值用数组
- **日期**：ISO 格式（YYYY-MM-DD）
- **数字**：用于计数、评分、优先级
- **复选框**：用于布尔状态

### 质量检查

- ✅ YAML 语法有效
- ✅ 没重复属性
- ✅ 类型合适
- ✅ 内部链接加引号
- ✅ 值有意义（不空）

## 特殊情况

### 没标题的笔记

从下面生成标题：

1. 第一个 heading（如果有）
2. 第一段总结
3. 主要讨论的话题 / 概念

### 批量处理

处理文件夹时：

- 同类笔记保持一致
- 同概念用同属性名
- 报告变更总结

### 已有属性

- 保留有效的已有属性
- 更新废弃格式
- 小心合并新属性
- 没理由不删

## 例子

### 改前：

```markdown
今天和团队开了一个不错的会，讨论 Q1 规划...
```

### 改后：

```markdown
---
title: Q1 Planning Team Meeting
date: 2025-09-02
type: meeting
attendees: ['Team']
project: Q1 Planning
tags: [meeting, planning, q1-2025]
status: complete
---

今天和团队开了一个不错的会，讨论 Q1 规划...
```

记住：属性应该增强组织，而不是制造杂乱。只加那些有助于查找和连接笔记的内容。
