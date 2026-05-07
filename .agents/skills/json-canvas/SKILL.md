---
name: json-canvas
description: Create and edit JSON Canvas files (.canvas) with nodes, edges, groups, and connections. Use when working with .canvas files, creating visual canvases, mind maps, flowcharts, or when the user mentions Canvas files in Obsidian. / 创建和编辑 JSON Canvas 文件（.canvas），含 node、edge、group、connection。当处理 .canvas 文件、做可视化画布 / 思维导图 / 流程图，或用户提到 Obsidian 的 Canvas 文件时触发。
---

# JSON Canvas Skill

让兼容 skill 的 agent 能创建和编辑有效的 JSON Canvas 文件（`.canvas`），用于 Obsidian 和其他应用。

## 概览

JSON Canvas 是开放的无限画布数据文件格式。Canvas 文件用 `.canvas` 扩展名，含合法 JSON，遵循 [JSON Canvas Spec 1.0](https://jsoncanvas.org/spec/1.0/)。

## 文件结构

canvas 文件含两个顶层数组：

```json
{
  "nodes": [],
  "edges": []
}
```

- `nodes`（可选）：node 对象数组
- `edges`（可选）：连接 node 的 edge 对象数组

## Nodes

node 是画布上的对象。四种 node 类型：
- `text` — 含 Markdown 的文本内容
- `file` — 引用文件 / 附件
- `link` — 外部 URL
- `group` — 包含其他 node 的可视容器

### Z-Index 排序

node 在数组里的顺序决定 z-index：
- 第一个 node = 底层（在其他下面）
- 最后一个 node = 顶层（在其他上面）

### 通用 node 属性

所有 node 共享这些属性：

| 属性 | 必需 | 类型 | 描述 |
|-----------|----------|------|-------------|
| `id` | 是 | string | node 的唯一标识 |
| `type` | 是 | string | node 类型：`text`、`file`、`link`、`group` |
| `x` | 是 | integer | X 位置（像素） |
| `y` | 是 | integer | Y 位置（像素） |
| `width` | 是 | integer | 宽（像素） |
| `height` | 是 | integer | 高（像素） |
| `color` | 否 | canvasColor | node 颜色（见 Color 段） |

### Text Node

text node 含 Markdown 内容。

```json
{
  "id": "6f0ad84f44ce9c17",
  "type": "text",
  "x": 0,
  "y": 0,
  "width": 400,
  "height": 200,
  "text": "# Hello World\n\nThis is **Markdown** content."
}
```

| 属性 | 必需 | 类型 | 描述 |
|-----------|----------|------|-------------|
| `text` | 是 | string | 含 Markdown 语法的纯文本 |

### File Node

file node 引用文件或附件（图片、视频、PDF、笔记等）。

```json
{
  "id": "a1b2c3d4e5f67890",
  "type": "file",
  "x": 500,
  "y": 0,
  "width": 400,
  "height": 300,
  "file": "Attachments/diagram.png"
}
```

```json
{
  "id": "b2c3d4e5f6789012",
  "type": "file",
  "x": 500,
  "y": 400,
  "width": 400,
  "height": 300,
  "file": "Notes/Project Overview.md",
  "subpath": "#Implementation"
}
```

| 属性 | 必需 | 类型 | 描述 |
|-----------|----------|------|-------------|
| `file` | 是 | string | 系统内文件路径 |
| `subpath` | 否 | string | 链接到 heading 或 block（以 `#` 开头） |

### Link Node

link node 显示外部 URL。

```json
{
  "id": "c3d4e5f678901234",
  "type": "link",
  "x": 1000,
  "y": 0,
  "width": 400,
  "height": 200,
  "url": "https://obsidian.md"
}
```

| 属性 | 必需 | 类型 | 描述 |
|-----------|----------|------|-------------|
| `url` | 是 | string | 外部 URL |

### Group Node

group node 是组织其他 node 的可视容器。

```json
{
  "id": "d4e5f6789012345a",
  "type": "group",
  "x": -50,
  "y": -50,
  "width": 1000,
  "height": 600,
  "label": "Project Overview",
  "color": "4"
}
```

```json
{
  "id": "e5f67890123456ab",
  "type": "group",
  "x": 0,
  "y": 700,
  "width": 800,
  "height": 500,
  "label": "Resources",
  "background": "Attachments/background.png",
  "backgroundStyle": "cover"
}
```

| 属性 | 必需 | 类型 | 描述 |
|-----------|----------|------|-------------|
| `label` | 否 | string | group 文本标签 |
| `background` | 否 | string | 背景图路径 |
| `backgroundStyle` | 否 | string | 背景渲染样式 |

#### Background 样式

| 值 | 描述 |
|-------|-------------|
| `cover` | 填满 node 整个宽高 |
| `ratio` | 保持背景图纵横比 |
| `repeat` | 双向重复图案 |

## Edges

edge 是连接 node 的线。

```json
{
  "id": "f67890123456789a",
  "fromNode": "6f0ad84f44ce9c17",
  "toNode": "a1b2c3d4e5f67890"
}
```

```json
{
  "id": "0123456789abcdef",
  "fromNode": "6f0ad84f44ce9c17",
  "fromSide": "right",
  "fromEnd": "none",
  "toNode": "b2c3d4e5f6789012",
  "toSide": "left",
  "toEnd": "arrow",
  "color": "1",
  "label": "leads to"
}
```

| 属性 | 必需 | 类型 | 默认 | 描述 |
|-----------|----------|------|---------|-------------|
| `id` | 是 | string | - | edge 的唯一标识 |
| `fromNode` | 是 | string | - | 连接起点 node ID |
| `fromSide` | 否 | string | - | edge 起点边 |
| `fromEnd` | 否 | string | `none` | edge 起点形状 |
| `toNode` | 是 | string | - | 连接终点 node ID |
| `toSide` | 否 | string | - | edge 终点边 |
| `toEnd` | 否 | string | `arrow` | edge 终点形状 |
| `color` | 否 | canvasColor | - | 线颜色 |
| `label` | 否 | string | - | edge 文本标签 |

### Side 值

| 值 | 描述 |
|-------|-------------|
| `top` | node 上边 |
| `right` | node 右边 |
| `bottom` | node 下边 |
| `left` | node 左边 |

### 端点形状

| 值 | 描述 |
|-------|-------------|
| `none` | 无端点形状 |
| `arrow` | 箭头端点 |

## 颜色

`canvasColor` 类型有两种指定方式：

### Hex 颜色

```json
{
  "color": "#FF0000"
}
```

### 预设颜色

```json
{
  "color": "1"
}
```

| 预设 | 颜色 |
|--------|-------|
| `"1"` | 红 |
| `"2"` | 橙 |
| `"3"` | 黄 |
| `"4"` | 绿 |
| `"5"` | 青 |
| `"6"` | 紫 |

注意：预设的具体颜色值有意未定义，让应用能用自己的品牌色。

## 完整示例

### 简单 Canvas（文本 + 连线）

```json
{
  "nodes": [
    {
      "id": "8a9b0c1d2e3f4a5b",
      "type": "text",
      "x": 0,
      "y": 0,
      "width": 300,
      "height": 150,
      "text": "# Main Idea\n\nThis is the central concept."
    },
    {
      "id": "1a2b3c4d5e6f7a8b",
      "type": "text",
      "x": 400,
      "y": -100,
      "width": 250,
      "height": 100,
      "text": "## Supporting Point A\n\nDetails here."
    },
    {
      "id": "2b3c4d5e6f7a8b9c",
      "type": "text",
      "x": 400,
      "y": 100,
      "width": 250,
      "height": 100,
      "text": "## Supporting Point B\n\nMore details."
    }
  ],
  "edges": [
    {
      "id": "3c4d5e6f7a8b9c0d",
      "fromNode": "8a9b0c1d2e3f4a5b",
      "fromSide": "right",
      "toNode": "1a2b3c4d5e6f7a8b",
      "toSide": "left"
    },
    {
      "id": "4d5e6f7a8b9c0d1e",
      "fromNode": "8a9b0c1d2e3f4a5b",
      "fromSide": "right",
      "toNode": "2b3c4d5e6f7a8b9c",
      "toSide": "left"
    }
  ]
}
```

### 带 Group 的项目看板

```json
{
  "nodes": [
    {
      "id": "5e6f7a8b9c0d1e2f",
      "type": "group",
      "x": 0,
      "y": 0,
      "width": 300,
      "height": 500,
      "label": "To Do",
      "color": "1"
    },
    {
      "id": "6f7a8b9c0d1e2f3a",
      "type": "group",
      "x": 350,
      "y": 0,
      "width": 300,
      "height": 500,
      "label": "In Progress",
      "color": "3"
    },
    {
      "id": "7a8b9c0d1e2f3a4b",
      "type": "group",
      "x": 700,
      "y": 0,
      "width": 300,
      "height": 500,
      "label": "Done",
      "color": "4"
    },
    {
      "id": "8b9c0d1e2f3a4b5c",
      "type": "text",
      "x": 20,
      "y": 50,
      "width": 260,
      "height": 80,
      "text": "## Task 1\n\nImplement feature X"
    },
    {
      "id": "9c0d1e2f3a4b5c6d",
      "type": "text",
      "x": 370,
      "y": 50,
      "width": 260,
      "height": 80,
      "text": "## Task 2\n\nReview PR #123",
      "color": "2"
    },
    {
      "id": "0d1e2f3a4b5c6d7e",
      "type": "text",
      "x": 720,
      "y": 50,
      "width": 260,
      "height": 80,
      "text": "## Task 3\n\n~~Setup CI/CD~~"
    }
  ],
  "edges": []
}
```

### 含文件和链接的研究 Canvas

```json
{
  "nodes": [
    {
      "id": "1e2f3a4b5c6d7e8f",
      "type": "text",
      "x": 300,
      "y": 200,
      "width": 400,
      "height": 200,
      "text": "# Research Topic\n\n## Key Questions\n\n- How does X affect Y?\n- What are the implications?",
      "color": "5"
    },
    {
      "id": "2f3a4b5c6d7e8f9a",
      "type": "file",
      "x": 0,
      "y": 0,
      "width": 250,
      "height": 150,
      "file": "Literature/Paper A.pdf"
    },
    {
      "id": "3a4b5c6d7e8f9a0b",
      "type": "file",
      "x": 0,
      "y": 200,
      "width": 250,
      "height": 150,
      "file": "Notes/Meeting Notes.md",
      "subpath": "#Key Insights"
    },
    {
      "id": "4b5c6d7e8f9a0b1c",
      "type": "link",
      "x": 0,
      "y": 400,
      "width": 250,
      "height": 100,
      "url": "https://example.com/research"
    },
    {
      "id": "5c6d7e8f9a0b1c2d",
      "type": "file",
      "x": 750,
      "y": 150,
      "width": 300,
      "height": 250,
      "file": "Attachments/diagram.png"
    }
  ],
  "edges": [
    {
      "id": "6d7e8f9a0b1c2d3e",
      "fromNode": "2f3a4b5c6d7e8f9a",
      "fromSide": "right",
      "toNode": "1e2f3a4b5c6d7e8f",
      "toSide": "left",
      "label": "supports"
    },
    {
      "id": "7e8f9a0b1c2d3e4f",
      "fromNode": "3a4b5c6d7e8f9a0b",
      "fromSide": "right",
      "toNode": "1e2f3a4b5c6d7e8f",
      "toSide": "left",
      "label": "informs"
    },
    {
      "id": "8f9a0b1c2d3e4f5a",
      "fromNode": "4b5c6d7e8f9a0b1c",
      "fromSide": "right",
      "toNode": "1e2f3a4b5c6d7e8f",
      "toSide": "left",
      "toEnd": "arrow",
      "color": "6"
    },
    {
      "id": "9a0b1c2d3e4f5a6b",
      "fromNode": "1e2f3a4b5c6d7e8f",
      "fromSide": "right",
      "toNode": "5c6d7e8f9a0b1c2d",
      "toSide": "left",
      "label": "visualized by"
    }
  ]
}
```

### 流程图

```json
{
  "nodes": [
    {
      "id": "a0b1c2d3e4f5a6b7",
      "type": "text",
      "x": 200,
      "y": 0,
      "width": 150,
      "height": 60,
      "text": "**Start**",
      "color": "4"
    },
    {
      "id": "b1c2d3e4f5a6b7c8",
      "type": "text",
      "x": 200,
      "y": 100,
      "width": 150,
      "height": 60,
      "text": "Step 1:\nGather data"
    },
    {
      "id": "c2d3e4f5a6b7c8d9",
      "type": "text",
      "x": 200,
      "y": 200,
      "width": 150,
      "height": 80,
      "text": "**Decision**\n\nIs data valid?",
      "color": "3"
    },
    {
      "id": "d3e4f5a6b7c8d9e0",
      "type": "text",
      "x": 400,
      "y": 200,
      "width": 150,
      "height": 60,
      "text": "Process data"
    },
    {
      "id": "e4f5a6b7c8d9e0f1",
      "type": "text",
      "x": 0,
      "y": 200,
      "width": 150,
      "height": 60,
      "text": "Request new data",
      "color": "1"
    },
    {
      "id": "f5a6b7c8d9e0f1a2",
      "type": "text",
      "x": 400,
      "y": 320,
      "width": 150,
      "height": 60,
      "text": "**End**",
      "color": "4"
    }
  ],
  "edges": [
    {
      "id": "a6b7c8d9e0f1a2b3",
      "fromNode": "a0b1c2d3e4f5a6b7",
      "fromSide": "bottom",
      "toNode": "b1c2d3e4f5a6b7c8",
      "toSide": "top"
    },
    {
      "id": "b7c8d9e0f1a2b3c4",
      "fromNode": "b1c2d3e4f5a6b7c8",
      "fromSide": "bottom",
      "toNode": "c2d3e4f5a6b7c8d9",
      "toSide": "top"
    },
    {
      "id": "c8d9e0f1a2b3c4d5",
      "fromNode": "c2d3e4f5a6b7c8d9",
      "fromSide": "right",
      "toNode": "d3e4f5a6b7c8d9e0",
      "toSide": "left",
      "label": "Yes",
      "color": "4"
    },
    {
      "id": "d9e0f1a2b3c4d5e6",
      "fromNode": "c2d3e4f5a6b7c8d9",
      "fromSide": "left",
      "toNode": "e4f5a6b7c8d9e0f1",
      "toSide": "right",
      "label": "No",
      "color": "1"
    },
    {
      "id": "e0f1a2b3c4d5e6f7",
      "fromNode": "e4f5a6b7c8d9e0f1",
      "fromSide": "top",
      "fromEnd": "none",
      "toNode": "b1c2d3e4f5a6b7c8",
      "toSide": "left",
      "toEnd": "arrow"
    },
    {
      "id": "f1a2b3c4d5e6f7a8",
      "fromNode": "d3e4f5a6b7c8d9e0",
      "fromSide": "bottom",
      "toNode": "f5a6b7c8d9e0f1a2",
      "toSide": "top"
    }
  ]
}
```

## ID 生成

node 和 edge 的 ID 必须是唯一字符串。Obsidian 生成 16 位十六进制 ID：

```json
"id": "6f0ad84f44ce9c17"
"id": "a3b2c1d0e9f8g7h6"
"id": "1234567890abcdef"
```

格式是 16 位小写 hex 字符串（64 位随机值）。

## 布局指南

### 定位

- 坐标可以是负的（画布无限延伸）
- `x` 向右增大
- `y` 向下增大
- 位置指 node 左上角

### 推荐尺寸

| Node 类型 | 建议宽 | 建议高 |
|-----------|-----------------|------------------|
| 小文本 | 200-300 | 80-150 |
| 中文本 | 300-450 | 150-300 |
| 大文本 | 400-600 | 300-500 |
| 文件预览 | 300-500 | 200-400 |
| 链接预览 | 250-400 | 100-200 |
| Group | 视情况 | 视情况 |

### 间距

- group 内部留 20-50px padding
- node 之间 50-100px 间距，可读性好
- 对齐到 grid（10 或 20 的倍数），布局更整洁

## 校验规则

1. 所有 `id` 在 node 和 edge 间都必须唯一
2. `fromNode` 和 `toNode` 必须引用已存在的 node ID
3. 每种 node 类型的必需字段都要在
4. `type` 必须是：`text`、`file`、`link`、`group` 之一
5. `backgroundStyle` 必须是：`cover`、`ratio`、`repeat` 之一
6. `fromSide`、`toSide` 必须是：`top`、`right`、`bottom`、`left` 之一
7. `fromEnd`、`toEnd` 必须是：`none`、`arrow` 之一
8. 颜色预设必须是 `"1"` 到 `"6"` 或合法 hex 颜色

## 参考

- [JSON Canvas Spec 1.0](https://jsoncanvas.org/spec/1.0/)
- [JSON Canvas GitHub](https://github.com/obsidianmd/jsoncanvas)
