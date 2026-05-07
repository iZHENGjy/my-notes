---
name: download-attachment
description: Download files from URLs to the Obsidian attachments folder and organize them with descriptive names. Use when the user provides URLs to download, asks to save files from the web, or wants to add web content as attachments. / 从 URL 下载文件到 Obsidian 附件目录，并用描述性名字整理。当用户给 URL 让下载、要从网上保存文件、或要把网页内容加为附件时触发。
---

# download-attachment

从 URL 下载文件到附件目录，并用描述性名字整理。

## 用法

```
/download-attachment <url1> [url2] [url3...]
```

## 示例

```
/download-attachment https://example.com/document.pdf
/download-attachment https://site.com/image.png https://site.com/report.pdf
```

## 实现

任务：从 URL 下载文件，整理到 Obsidian vault 附件目录。

### Step 1：解析并校验 URL

从用户输入抽取 URL，支持多个。

- **校验 URL scheme**：只允许 http:// 或 https://
- **拒绝非法 URL**：file://、ftp:// 或畸形 URL
- **校验示例**：

```bash
if [[ ! "$url" =~ ^https?:// ]]; then
  echo "Error: Only HTTP/HTTPS URLs are allowed"
  exit 1
fi
```

### Step 2：下载文件

每个 URL：

```bash
# 文件名清洗，防路径穿越
# 去掉 ../ 和其他危险字符
filename=$(basename "$url" | sed 's/[^a-zA-Z0-9._-]/_/g')

# 用 wget 或 curl 下载，带 timeout
wget --timeout=30 -O "05_Attachments/$filename" "$url"
# 或
curl --max-time 30 -L "$url" -o "05_Attachments/$filename"
```

### Step 3：验证下载

确认文件下载成功：

```bash
ls -la "05_Attachments/"
```

### Step 4：整理文件

下载完跑 organize-attachments 命令，给文件起描述性名字：

PDF：

- 用 `pdftotext` 抽文本
- 分析内容，找出有意义的标题

图片：

- 用 `mcp__gemini-vision__analyze_image` 或 `mcp__gemini-vision__analyze_multiple`
- 基于内容生成描述性文件名

### Step 5：移到 Organized

把重命名后的文件移到 `05_Attachments/Organized/`，用描述性名字。

### Step 6：更新索引

在 `05_Attachments/00_Index.md` 加条目。

### Step 7：提交变更

```bash
git add -A
git commit -m "Download and organize attachments from URLs"
git push
```

## 注意事项

1. **文件命名**：
   - 初次下载：用 URL 文件名或从 URL 生成
   - 分析后：用描述性标题重命名

2. **支持的类型**：
   - 图片：.png、.jpg、.jpeg、.gif、.webp
   - 文档：.pdf、.doc、.docx
   - 文本：.txt、.md
   - 数据：.csv、.xlsx

3. **错误处理**：
   - 检查 URL 是否可达
   - 验证文件下载完整
   - 优雅处理下载失败

4. **组织**：
   - 下载文件先到 `05_Attachments/`
   - 重命名后移到 `05_Attachments/Organized/`
   - 必要时更新 vault 里的链接

## 工作流

1. 从给定 URL 下载文件
2. 识别文件类型并分析内容
3. 生成描述性文件名
4. 移到 Organized 文件夹
5. 更新索引和引用
6. Commit 并 push

## 几条建议

- 多 URL 批处理更高效
- 图片批量分析用 Gemini Vision（一次最多 3 张）
- PDF 重命名前先抽取有意义的上下文
- 保留原始扩展名
- 文件名简洁但描述性强（最多 60 字符）
