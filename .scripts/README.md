# Scripts 目录

Vault 自动化和网页内容抓取的辅助脚本。

## 可用脚本

### 附件管理

主要通过 package.json 里的 npm/pnpm 命令调用：

- `update-attachment-links.js` — 移动附件后更新笔记里的链接
- `fix-renamed-links.js` — 重命名文件后修复链接

### 网页内容抓取

**注意**：这些脚本需要 API key 才能跑：

#### firecrawl-scrape.sh

抓取单个 URL，存为 markdown。

```bash
# 需要环境变量 FIRECRAWL_API_KEY
.scripts/firecrawl-scrape.sh <url> <output_file>
```

#### firecrawl-batch.sh

批量抓多个 URL，自动生成文件名。

```bash
# 需要环境变量 FIRECRAWL_API_KEY

# 基本用法 — 默认存到 00_Inbox/Clippings/
.scripts/firecrawl-batch.sh <url1> <url2> <url3>

# 自定义输出目录
.scripts/firecrawl-batch.sh -o 01_Projects/Research/ <url1> <url2>
.scripts/firecrawl-batch.sh --output-dir 03_Resources/Articles/ <url1> <url2>
```

### 字幕提取

#### transcript-extract.sh

从 YouTube 视频提取字幕。

```bash
# 基本用法 — 默认存到 00_Inbox/Clippings/
.scripts/transcript-extract.sh <youtube-url>

# 自定义输出目录
.scripts/transcript-extract.sh <youtube-url> 01_Projects/Research/
```

## NPM 脚本

在 vault 根目录用 `pnpm` 跑：

| 命令                           | 说明                                    |
| ------------------------------ | -------------------------------------- |
| `attachments:list`             | 列出前 20 个未处理附件                 |
| `attachments:count`            | 统计未处理附件数                       |
| `attachments:organized`        | 统计 Organized 文件夹里的文件数        |
| `attachments:unprocessed`      | 同 count                               |
| `attachments:refs <file>`      | 找指定文件的引用                       |
| `attachments:sizes`            | 显示前 20 大附件                       |
| `attachments:orphans`          | 找出无引用的附件                       |
| `attachments:recent`           | 显示近 7 天新增文件                    |
| `attachments:create-organized` | 创建 Organized 子目录                  |

## 安装要求

### 网页抓取

1. 在 [firecrawl.dev](https://firecrawl.dev) 拿一个 Firecrawl API key
2. 加到 shell profile：
   ```bash
   export FIRECRAWL_API_KEY="your-key-here"
   ```

### 字幕提取

- 需要装 `yt-dlp` 和 `jq`：

  ```bash
  # macOS
  brew install yt-dlp jq

  # Linux
  apt-get install yt-dlp jq
  ```

## 添加自定义脚本

1. 在 `.scripts/` 里写脚本
2. 加可执行权限：`chmod +x .scripts/your-script.sh`
3. 必要时在 `package.json` 加 npm script
4. 在这里写文档

## 注意

- 脚本默认是 Unix 环境（macOS / Linux）
- Windows 用户可能需要 WSL 或 Git Bash
- 所有路径相对于 vault 根目录
- 其他要求看脚本里的注释
