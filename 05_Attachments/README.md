# 📎 Attachments

存图片、PDF 和其他非文本文件。

## 干啥用的

集中存放：
- 图片和截图
- PDF 和文档
- 表格和数据文件
- 音频和视频
- 笔记里引用的所有二进制文件

## 组织方式

```
05_Attachments/
├── Organized/          # 已处理、命名规范的文件
│   ├── Images/
│   ├── PDFs/
│   └── Data/
├── IMG_*.png          # 没处理的手机照片
├── Screenshot*.png    # 没处理的截图
├── CleanShot*.png    # 没处理的 CleanShot 文件
└── *.pdf             # 各种 PDF
```

## 命名规范

### 处理前
- `IMG_1234.png`（手机出来的）
- `Screenshot 2024-03-15 at 2.30.45 PM.png`
- `CleanShot 2024-03-15 at 14.30.45.png`
- `document(1).pdf`

### 处理后
- `2024-03-15_Project_Architecture_Diagram.png`
- `2024-03-15_Meeting_Whiteboard.jpg`
- `API_Documentation_v2.pdf`
- `Customer_Interview_Transcript.pdf`

## 辅助脚本

用 `pnpm` 跑：

### 查看状态
- `attachments:list` — 列出未处理文件
- `attachments:count` — 统计未处理文件数
- `attachments:organized` — 统计已整理的文件数
- `attachments:sizes` — 显示最大的文件
- `attachments:recent` — 近 7 天新增的文件

### 找问题
- `attachments:orphans` — 没有任何笔记引用的文件
- `attachments:refs [filename]` — 找谁在引用某文件

### 整理
- `attachments:create-organized` — 建 Organized 文件夹

## Claude Code 工作流

### 处理截图
```
看下 05_Attachments 里最近的截图。
根据内容建议更好的命名。
帮我整理一下。
```

### 找孤儿文件
```
找出没有被任何笔记引用的所有附件。
有哪些可以删？
```

### 批量重命名
```
看 Attachments 里没处理的图片。
基于内容建议描述性名字。
```

### 清理
```
找出 Attachments 里的重复图片。
找出超过 10MB 的文件。
什么可以压缩或删除？
```

## 最佳实践

### 文件大小
- 图片控制在 2MB 以下（Git 友好）
- 大 PDF 压缩
- 视频用外部存储
- 提交前优化图片

### 命名
- 加日期：`YYYY-MM-DD`
- 描述性但简洁
- 用下划线不用空格
- 必要时加版本号

### 链接
```markdown
# 嵌入图片
![[05_Attachments/Organized/diagram.png]]

# 链接 PDF
[[05_Attachments/Organized/document.pdf]]

# 加描述
![[05_Attachments/Organized/chart.png|Q1 销售图]]
```

## 处理流程

1. **捕获**：把文件存到 `05_Attachments/`
2. **审视**：看下内容，确定用途
3. **重命名**：起描述性、带日期的名字
4. **整理**：移到 `Organized/` 子目录
5. **链接**：更新笔记里的引用
6. **清理**：删除孤儿文件

## Claude Code 提示词

### Vision 分析
```
分析下 Attachments 里的图片。
内容是什么？
建议合适的名字和组织方式。
```

### 批量处理
```
处理本周所有 CleanShot 文件。
基于内容重命名。
移到 Organized。
```

### 存储审计
```
分析附件存储：
- 总大小
- 最大文件
- 文件类型分布
- 孤儿文件
```

## 几条建议

- **每周处理** — 别让文件堆起来
- **立刻命名** — 上下文消失得很快
- **有目的地链接** — 只嵌入有价值的
- **大胆压缩** — 存储空间会越占越多
- **果断删** — 不是每张截图都重要

## Git 注意事项

### 建议的 .gitignore
```
*.mp4
*.mov
*.zip
.DS_Store
files_over_10mb/
```

### 大文件处理
- 超过 10MB 用 Git LFS
- 考虑外部存储
- 改成链接到云存储
- 本地保留但 gitignore

## 记住

附件是用来支持笔记的，不是替代笔记。一个命名规范、组织良好的附件，胜过一千张随手截图。让 Claude Code 的 vision 能力帮你高效处理和组织视觉内容。
