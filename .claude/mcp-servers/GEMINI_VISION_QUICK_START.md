# Gemini Vision MCP Server — 快速上手

**新机器上 5 分钟内跑起来**

## 前置检查

跑这些命令确认环境齐活：

```bash
node --version  # 应该 v22+
pnpm --version  # 应该已安装
claude --version  # Claude Code 应该已安装
```

如果缺哪个：

- Node.js：从 [nodejs.org](https://nodejs.org/) 装（v22+）
- pnpm：`npm install -g pnpm`
- Claude Code：从 [claude.ai/code](https://claude.ai/code) 下载

## Step 1：拿 Gemini API Key

1. 访问 [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. 点 "Create API Key"
3. 复制 key（`AIzaSy...` 开头）

## Step 2：设置环境变量

### Linux/macOS Bash：

```bash
echo 'export GEMINI_API_KEY="your-actual-api-key-here"' >> ~/.bashrc
source ~/.bashrc
echo $GEMINI_API_KEY  # 验证一下能看到 key
```

### Linux/macOS Zsh：

```bash
echo 'export GEMINI_API_KEY="your-actual-api-key-here"' >> ~/.zshrc
source ~/.zshrc
echo $GEMINI_API_KEY  # 验证一下能看到 key
```

### Windows PowerShell：

```powershell
[System.Environment]::SetEnvironmentVariable('GEMINI_API_KEY', 'your-key-here', 'User')
# 重启 PowerShell
$env:GEMINI_API_KEY  # 验证一下能看到 key
```

## Step 3：装依赖

**⚠️ 关键：这一步必须在加 MCP server 之前做！**

进 Obsidian vault 目录：

```bash
cd ~/dev/02_Areas/Obsidian  # 或你自己 vault 的位置
```

装依赖：

```bash
# 装 npm 包（必须先做！）
pnpm install

# 这一步会装：
# - @google/generative-ai（Gemini API 客户端）
# - @modelcontextprotocol/sdk（MCP server 框架）
# - package.json 里的其他依赖
```

**常见错误**：如果看到
`Error [ERR_MODULE_NOT_FOUND]: Cannot find package '@modelcontextprotocol/sdk'`，
说明你忘跑 `pnpm install` 了！

**让 Obsidian 隐藏 node_modules**（可选但推荐）：

1. 打开 Obsidian
2. Settings → Files & Links → Excluded files
3. 点 "Manage"
4. 把 `node_modules/` 加进列表
5. 也可以加：`pnpm-lock.yaml`、`.gitignore`

这样 vault 看起来干净，同时还能用标准的 Node.js 模块解析。

## Step 4：注册 MCP Server

**项目级安装（推荐用于团队协作）：**

```bash
# 加到项目（会创建 .mcp.json）
claude mcp add --scope project gemini-vision node .claude/mcp-servers/gemini-vision.mjs
```

**用户级安装（个人跨项目用）：**

```bash
# 加到用户配置
claude mcp add --scope user gemini-vision node .claude/mcp-servers/gemini-vision.mjs
```

加完后需要编辑 `.mcp.json` 加 API key：

```json
{
  "mcpServers": {
    "gemini-vision": {
      "type": "stdio",
      "command": "node",
      "args": [".claude/mcp-servers/gemini-vision.mjs"],
      "env": {
        "GEMINI_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

**注意**：

- 命令必须在 Obsidian vault 根目录跑
- 必须先跑过 `pnpm install`
- `.mcp.json` 已 gitignore 防止泄露

## Step 5：验证能用

1. **打开一个新的 Claude Code 窗口**（必须是新的）：

   ```bash
   cd ~/dev/Obsidian
   claude
   ```

2. **看 server 是否连上**：在 Claude 里输入 `/mcp`

   应该看到：

   ```
   gemini-vision ✔ connected
   ```

3. **真跑一条命令试**：
   ```
   用 gemini-vision 从 05_Attachments/[任意图片.png] 提取文字
   ```

## 排错

### "gemini-vision failed" 或 /mcp 里看不到

1. **最常见 — 依赖没装**：

   ```bash
   # 如果看到：Error [ERR_MODULE_NOT_FOUND]: Cannot find package '@modelcontextprotocol/sdk'
   # 跑这个：
   pnpm install
   ```

   然后在 Claude Code 里重连 MCP server。

2. **检查 API key 配置**：
   - 项目级：检查 `.mcp.json` 的 env 段里有 API key
   - 用户级：检查 `~/.claude.json` 里有 API key
   - 格式应该是：`"GEMINI_API_KEY": "AIzaSy..."`

3. **直接跑 server 测**：

   ```bash
   export GEMINI_API_KEY="your-api-key-here"
   node .claude/mcp-servers/gemini-vision.mjs
   ```

   应该看到："🚀 Gemini Vision MCP Server running"。Ctrl+C 退出。

4. **重新加 server（项目级）**：

   ```bash
   claude mcp remove gemini-vision --scope project
   claude mcp add --scope project gemini-vision node .claude/mcp-servers/gemini-vision.mjs
   # 然后编辑 .mcp.json 加 API key
   ```

5. **看日志**：

   ```bash
   # 找日志目录
   ls ~/Library/Caches/claude-cli-nodejs/*/mcp-logs-gemini-vision/
   # 或 Linux：
   ls ~/.cache/claude-cli-nodejs/*/mcp-logs-gemini-vision/

   # 看最新日志
   tail -f [log-directory]/*.txt
   ```

### "Cannot find module" 错误

1. **确认 package.json 在**：

   ```bash
   cat package.json
   ```

   应该能看到 @google/generative-ai 和 @modelcontextprotocol/sdk

2. **重装依赖**：

   ```bash
   rm -rf node_modules pnpm-lock.yaml
   pnpm install
   ```

3. **确认 node_modules 已生成**：
   ```bash
   ls node_modules/@google/generative-ai
   ```

### Server 起来了但工具不工作

1. **直接测 API key**：

   ```bash
   curl "https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY"
   ```

   应该返回模型列表，不是错误。

2. **检查文件路径**：
   - 用相对于 vault 根的绝对路径
   - 例：`05_Attachments/image.png`，不要 `./05_Attachments/image.png`

## 可用工具

跑起来后，在 Claude 里能用这些：

### 图像分析

```
# 分析图片
用 gemini-vision 分析 05_Attachments/screenshot.png

# 提取文字（OCR）
用 gemini-vision 从 05_Attachments/document.jpg 提取文字

# 对比图片
用 gemini-vision 对比 image1.png 和 image2.png

# 建议文件名
用 gemini-vision 给 IMG_1234.jpg 建议文件名

# 多图分析
用 gemini-vision 分析多张：image1.png、image2.png、image3.png
```

### 视频分析（新！）

```
# 分析本地视频
用 gemini-vision 分析视频 05_Attachments/video.mp4

# 分析 YouTube 视频
用 gemini-vision 分析 YouTube 视频 https://www.youtube.com/watch?v=VIDEO_ID

# 自定义视频分析 prompt
用 gemini-vision 分析视频 file.mp4，把里面所有可见文字提出来
```

**注意**：视频处理可能要 30-60 秒，因为文件需要进入 ACTIVE 状态才能分析。Server 会自动等待并显示进度。

### 支持的格式

**图片**：JPG、JPEG、PNG、GIF、BMP、WebP
**视频**：MP4、AVI、MOV、WebM、MKV、WMV、FLV、3GP、M4V
**文档**：PDF、TXT、DOC、DOCX、ODT、RTF
**特殊**：YouTube URL（直接支持，不用下载）

## 快速重装（之前装过的话）

如果你 shell profile 里已经设了 API key：

```bash
cd ~/dev/Obsidian
git pull
pnpm install
claude mcp add gemini-vision \
  --scope local \
  --env GEMINI_API_KEY=$GEMINI_API_KEY \
  -- node .claude/mcp-servers/gemini-vision.mjs
```

然后开新的 Claude 窗口测试。

## 文件位置

- **Server 代码**：`.claude/mcp-servers/gemini-vision.mjs`
- **依赖**：`package.json`
- **本指南**：`.claude/mcp-servers/GEMINI_VISION_QUICK_START.md`

## 需要帮助？

1. 先看上面的排错段
2. 确认所有前置都装了
3. 确认你在 Obsidian vault 根目录
4. 确认环境变量里有 API key

---

_最后测试：2025 年 9 月_
