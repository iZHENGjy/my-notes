# MCP Servers

Model Context Protocol 服务器，用来扩展 Claude Code 的能力。

## Gemini Vision MCP

接入 Google Gemini 模型，加入图像和文档分析能力。

### 功能

- **图像分析**：描述、分析、提取图片里的文字
- **文档处理**：分析 PDF 和文档
- **多图对比**：一次比较多张图
- **OCR**：从图片里抽文字
- **智能文件名建议**：给图片生成描述性文件名

### 安装

1. **拿一个 Gemini API Key**
   - 访问：https://aistudio.google.com/apikey
   - 创建一个免费 API key

2. **加到环境变量**

   ```bash
   # 加到 ~/.zshrc 或 ~/.bashrc
   export GEMINI_API_KEY='your-key-here'

   # 重新加载 shell
   source ~/.zshrc
   ```

3. **装依赖**

   ```bash
   pnpm install
   ```

4. **测试**
   ```bash
   pnpm test-gemini
   ```

### 可用命令

配置好后，Claude Code 里就能用这些命令：

- `mcp__gemini-vision__analyze_image` — 分析单张图片
- `mcp__gemini-vision__analyze_multiple` — 比较多张图片
- `mcp__gemini-vision__extract_text` — OCR 文字提取
- `mcp__gemini-vision__compare_images` — 比较两张图片
- `mcp__gemini-vision__suggest_image_filename` — 生成描述性文件名
- `mcp__gemini-vision__analyze_document` — 分析 PDF 和文档

### 用法示例

**分析截图**

```
分析 05_Attachments/screenshot.png 里的图，
告诉我里面有什么。
```

**处理多张图**

```
对比 05_Attachments/Organized/ 里的所有图，
找出共同主题。
```

**提取文字**

```
从 05_Attachments/document.pdf
里提取所有文字。
```

**重命名图片**

```
基于内容，给 05_Attachments/ 里所有图
建议更好的名字。
```

### 排错

**"GEMINI_API_KEY not found"**

- 确认 key 已加到 shell profile
- 重启终端和 Claude Code

**"File not found"**

- 用绝对路径或相对于 vault 根的路径
- 检查文件权限

**速率限制**

- 免费版：每分钟 15 次请求
- 用得多的话考虑升级

## 加更多 MCP

1. 把 MCP server 文件放到 `.claude/mcp-servers/`
2. 在 Claude 设置里加配置
3. 在这里写文档
4. 加用法示例

## 资源

- [MCP 官方文档](https://modelcontextprotocol.io)
- [Gemini API 文档](https://ai.google.dev)
- [Claude Code MCP 指南](https://claude.ai/docs/mcp)
