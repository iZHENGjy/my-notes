---
name: obsidian-markdown
description: Create and edit Obsidian Flavored Markdown with wikilinks, embeds, callouts, properties, and other Obsidian-specific syntax. Use when working with .md files in Obsidian, or when the user mentions wikilinks, callouts, frontmatter, tags, embeds, or Obsidian notes. / 创建和编辑 Obsidian Flavored Markdown，含 wikilink、embed、callout、properties 和其他 Obsidian 专用语法。当处理 Obsidian 里的 .md 文件，或用户提到 wikilink、callout、frontmatter、tags、embed、Obsidian 笔记时触发。
---

# Obsidian Flavored Markdown Skill

让兼容 skill 的 agent 能创建和编辑有效的 Obsidian Flavored Markdown，包含所有 Obsidian 专用语法扩展。

## 概览

Obsidian 用了几种 Markdown 风格的组合：
- [CommonMark](https://commonmark.org/)
- [GitHub Flavored Markdown](https://github.github.com/gfm/)
- [LaTeX](https://www.latex-project.org/)（数学）
- Obsidian 专用扩展（wikilink、callout、embed 等）

## 基础格式

### 段落与换行

```markdown
This is a paragraph.

This is another paragraph (blank line between creates separate paragraphs).

For a line break within a paragraph, add two spaces at the end  
or use Shift+Enter.
```

### 标题

```markdown
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

### 文本格式

| 样式 | 语法 | 示例 | 输出 |
|-------|--------|---------|--------|
| 粗体 | `**text**` 或 `__text__` | `**Bold**` | **Bold** |
| 斜体 | `*text*` 或 `_text_` | `*Italic*` | *Italic* |
| 粗斜 | `***text***` | `***Both***` | ***Both*** |
| 删除线 | `~~text~~` | `~~Striked~~` | ~~Striked~~ |
| 高亮 | `==text==` | `==Highlighted==` | ==Highlighted== |
| 行内代码 | `` `code` `` | `` `code` `` | `code` |

### 转义格式

用反斜杠转义特殊字符：
```markdown
\*This won't be italic\*
\#This won't be a heading
1\. This won't be a list item
```

常需转义的字符：`\*`、`\_`、`\#`、`` \` ``、`\|`、`\~`

## 内部链接（Wikilink）

### 基础链接

```markdown
[[Note Name]]
[[Note Name.md]]
[[Note Name|Display Text]]
```

### 链接到标题

```markdown
[[Note Name#Heading]]
[[Note Name#Heading|Custom Text]]
[[#Heading in same note]]
[[##Search all headings in vault]]
```

### 链接到 Block

```markdown
[[Note Name#^block-id]]
[[Note Name#^block-id|Custom Text]]
```

在段落末尾加 `^block-id` 定义 block ID：
```markdown
This is a paragraph that can be linked to. ^my-block-id
```

列表和引用块的 block ID 写在单独一行：
```markdown
> This is a quote
> With multiple lines

^quote-id
```

### 搜索链接

```markdown
[[##heading]]     在所有标题里搜含 "heading"
[[^^block]]       在所有 block 里搜含 "block"
```

## Markdown 风格链接

```markdown
[Display Text](Note%20Name.md)
[Display Text](Note%20Name.md#Heading)
[Display Text](https://example.com)
[Note](obsidian://open?vault=VaultName&file=Note.md)
```

注意：Markdown 链接里空格必须 URL 编码为 `%20`。

## Embed（嵌入）

### 嵌入笔记

```markdown
![[Note Name]]
![[Note Name#Heading]]
![[Note Name#^block-id]]
```

### 嵌入图片

```markdown
![[image.png]]
![[image.png|640x480]]    宽 x 高
![[image.png|300]]        只指定宽（保持纵横比）
```

### 外部图片

```markdown
![Alt text](https://example.com/image.png)
![Alt text|300](https://example.com/image.png)
```

### 嵌入音频

```markdown
![[audio.mp3]]
![[audio.ogg]]
```

### 嵌入 PDF

```markdown
![[document.pdf]]
![[document.pdf#page=3]]
![[document.pdf#height=400]]
```

### 嵌入列表

```markdown
![[Note#^list-id]]
```

列表用 block ID 定义：
```markdown
- Item 1
- Item 2
- Item 3

^list-id
```

### 嵌入搜索结果

````markdown
```query
tag:#project status:done
```
````

## Callout

### 基础 callout

```markdown
> [!note]
> This is a note callout.

> [!info] Custom Title
> This callout has a custom title.

> [!tip] Title Only
```

### 可折叠 callout

```markdown
> [!faq]- Collapsed by default
> This content is hidden until expanded.

> [!faq]+ Expanded by default
> This content is visible but can be collapsed.
```

### 嵌套 callout

```markdown
> [!question] Outer callout
> > [!note] Inner callout
> > Nested content
```

### 支持的 callout 类型

| 类型 | 别名 | 描述 |
|------|---------|-------------|
| `note` | - | 蓝色，铅笔图标 |
| `abstract` | `summary`、`tldr` | 蓝绿色，剪贴板图标 |
| `info` | - | 蓝色，info 图标 |
| `todo` | - | 蓝色，复选框图标 |
| `tip` | `hint`、`important` | 青色，火焰图标 |
| `success` | `check`、`done` | 绿色，对勾图标 |
| `question` | `help`、`faq` | 黄色，问号 |
| `warning` | `caution`、`attention` | 橙色，警告图标 |
| `failure` | `fail`、`missing` | 红色，X 图标 |
| `danger` | `error` | 红色，闪电图标 |
| `bug` | - | 红色，bug 图标 |
| `example` | - | 紫色，列表图标 |
| `quote` | `cite` | 灰色，引号图标 |

### 自定义 callout（CSS）

```css
.callout[data-callout="custom-type"] {
  --callout-color: 255, 0, 0;
  --callout-icon: lucide-alert-circle;
}
```

## 列表

### 无序列表

```markdown
- Item 1
- Item 2
  - Nested item
  - Another nested
- Item 3

* Also works with asterisks
+ Or plus signs
```

### 有序列表

```markdown
1. First item
2. Second item
   1. Nested numbered
   2. Another nested
3. Third item

1) Alternative syntax
2) With parentheses
```

### 任务列表

```markdown
- [ ] Incomplete task
- [x] Completed task
- [ ] Task with sub-tasks
  - [ ] Subtask 1
  - [x] Subtask 2
```

## 引用

```markdown
> This is a blockquote.
> It can span multiple lines.
>
> And include multiple paragraphs.
>
> > Nested quotes work too.
```

## 代码

### 行内代码

```markdown
Use `backticks` for inline code.
Use double backticks for ``code with a ` backtick inside``.
```

### 代码块

````markdown
```
Plain code block
```

```javascript
// Syntax highlighted code block
function hello() {
  console.log("Hello, world!");
}
```

```python
# Python example
def greet(name):
    print(f"Hello, {name}!")
```
````

### 嵌套代码块

外层代码块用更多反引号或波浪号：

`````markdown
````markdown
Here's how to create a code block:
```js
console.log("Hello")
```
````
`````

## 表格

```markdown
| Header 1 | Header 2 | Header 3 |
|----------|----------|----------|
| Cell 1   | Cell 2   | Cell 3   |
| Cell 4   | Cell 5   | Cell 6   |
```

### 对齐

```markdown
| Left     | Center   | Right    |
|:---------|:--------:|---------:|
| Left     | Center   | Right    |
```

### 表格里用竖线

用反斜杠转义：
```markdown
| Column 1 | Column 2 |
|----------|----------|
| [[Link\|Display]] | ![[Image\|100]] |
```

## 数学（LaTeX）

### 行内数学

```markdown
This is inline math: $e^{i\pi} + 1 = 0$
```

### 块级数学

```markdown
$$
\begin{vmatrix}
a & b \\
c & d
\end{vmatrix} = ad - bc
$$
```

### 常用数学语法

```markdown
$x^2$              上标
$x_i$              下标
$\frac{a}{b}$      分数
$\sqrt{x}$         平方根
$\sum_{i=1}^{n}$   求和
$\int_a^b$         积分
$\alpha, \beta$    希腊字母
```

## 图表（Mermaid）

````markdown
```mermaid
graph TD
    A[Start] --> B{Decision}
    B -->|Yes| C[Do this]
    B -->|No| D[Do that]
    C --> E[End]
    D --> E
```
````

### 时序图

````markdown
```mermaid
sequenceDiagram
    Alice->>Bob: Hello Bob
    Bob-->>Alice: Hi Alice
```
````

### 图表里的链接

````markdown
```mermaid
graph TD
    A[Biology]
    B[Chemistry]
    A --> B
    class A,B internal-link;
```
````

## 脚注

```markdown
This sentence has a footnote[^1].

[^1]: This is the footnote content.

You can also use named footnotes[^note].

[^note]: Named footnotes still appear as numbers.

Inline footnotes are also supported.^[This is an inline footnote.]
```

## 注释

```markdown
This is visible %%but this is hidden%% text.

%%
This entire block is hidden.
It won't appear in reading view.
%%
```

## 水平线

```markdown
---
***
___
- - -
* * *
```

## Properties（Frontmatter）

Properties 用 YAML frontmatter，写在笔记开头：

```yaml
---
title: My Note Title
date: 2024-01-15
tags:
  - project
  - important
aliases:
  - My Note
  - Alternative Name
cssclasses:
  - custom-class
status: in-progress
rating: 4.5
completed: false
due: 2024-02-01T14:30:00
---
```

### Property 类型

| 类型 | 示例 |
|------|---------|
| 文本 | `title: My Title` |
| 数字 | `rating: 4.5` |
| 复选框 | `completed: true` |
| 日期 | `date: 2024-01-15` |
| 日期 + 时间 | `due: 2024-01-15T14:30:00` |
| 列表 | `tags: [one, two]` 或 YAML 列表 |
| 链接 | `related: "[[Other Note]]"` |

### 默认 properties

- `tags` — 笔记标签
- `aliases` — 笔记的别名
- `cssclasses` — 应用到笔记的 CSS 类

## 标签（Tag）

```markdown
#tag
#nested/tag
#tag-with-dashes
#tag_with_underscores

在 frontmatter 里:
---
tags:
  - tag1
  - nested/tag2
---
```

标签可以含：
- 字母（任何语言）
- 数字（不能开头）
- 下划线 `_`
- 连字符 `-`
- 正斜杠 `/`（用于嵌套）

## HTML 内容

Obsidian 支持 Markdown 内嵌 HTML：

```markdown
<div class="custom-container">
  <span style="color: red;">Colored text</span>
</div>

<details>
  <summary>Click to expand</summary>
  Hidden content here.
</details>

<kbd>Ctrl</kbd> + <kbd>C</kbd>
```

## 完整示例

````markdown
---
title: Project Alpha
date: 2024-01-15
tags:
  - project
  - active
status: in-progress
priority: high
---

# Project Alpha

## Overview

This project aims to [[improve workflow]] using modern techniques.

> [!important] Key Deadline
> The first milestone is due on ==January 30th==.

## Tasks

- [x] Initial planning
- [x] Resource allocation
- [ ] Development phase
  - [ ] Backend implementation
  - [ ] Frontend design
- [ ] Testing
- [ ] Deployment

## Technical Notes

The main algorithm uses the formula $O(n \log n)$ for sorting.

```python
def process_data(items):
    return sorted(items, key=lambda x: x.priority)
```

## Architecture

```mermaid
graph LR
    A[Input] --> B[Process]
    B --> C[Output]
    B --> D[Cache]
```

## Related Documents

- ![[Meeting Notes 2024-01-10#Decisions]]
- [[Budget Allocation|Budget]]
- [[Team Members]]

## References

For more details, see the official documentation[^1].

[^1]: https://example.com/docs

%%
Internal notes:
- Review with team on Friday
- Consider alternative approaches
%%
````

## 参考

- [Basic formatting syntax](https://help.obsidian.md/syntax)
- [Advanced formatting syntax](https://help.obsidian.md/advanced-syntax)
- [Obsidian Flavored Markdown](https://help.obsidian.md/obsidian-flavored-markdown)
- [Internal links](https://help.obsidian.md/links)
- [Embed files](https://help.obsidian.md/embeds)
- [Callouts](https://help.obsidian.md/callouts)
- [Properties](https://help.obsidian.md/properties)
