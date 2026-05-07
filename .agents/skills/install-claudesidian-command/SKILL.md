---
name: install-claudesidian-command
description: Install claudesidian shell command to launch Claude Code from anywhere. Use when the user wants to install a shell alias/launcher for their vault, or asks to set up the claudesidian command. / 装一个 claudesidian shell 命令，从任何地方启动 Claude Code 进 vault。当用户要"装 vault 启动器"、"装 shell alias"、"设置 claudesidian 命令"时触发。
---

# Install Claudesidian Command

创建一个 shell alias / function，让你能从任何地方运行 `claudesidian` 在 Claude Code 里打开你的 vault。

## 任务

装一个 shell 命令：

1. 切到你的 claudesidian vault 目录
2. 启动 Claude Code
3. 在终端任何目录都能用

类似一个 vault 的快捷启动器。

## 流程

### 1. **检测当前设置**

- 检查用户用什么 shell（bash / zsh / fish）
- 找到当前工作目录（vault 路径）
- 决定合适的配置文件

### 2. **创建命令**

命令是一个 alias，作用是：

- 切到 vault 目录：`cd /path/to/your/vault`
- 尝试恢复已有会话：`claude --resume 2>/dev/null`
- 没有就回退到新会话：`|| claude`
- 全部装到一行，路径正确转义：
  `(cd "/path/to/vault" && (claude --resume 2>/dev/null || claude))`

**重要**：路径必须正确转义，处理空格和特殊字符。

这样自动进入 resume 模式（如果有已存会话），否则启动新会话。

### 3. **装到 shell 配置**

把 alias 加到合适的配置文件：

- **Bash**：`~/.bashrc` 或 `~/.bash_profile`
- **Zsh**：`~/.zshrc`
- **Fish**：`~/.config/fish/config.fish`

### 4. **验证安装**

- 显示加进去的那行
- 提醒用户重新加载 shell 或 source 配置
- 给一个测试命令

## Shell 检测

检测用户的默认 shell，支持命令行覆盖：

```bash
# 检查 shell 是否作为参数传入（/install-claudesidian-command zsh）
if [ -n "$1" ]; then
  # 用户用参数提供 shell 类型
  SHELL_TYPE="$1"
else
  # 从 $SHELL 自动检测（用户默认 shell，不是当前 shell）
  SHELL_TYPE=$(basename "$SHELL")
fi

# 校验 shell 类型并设置合适的配置文件
case "$SHELL_TYPE" in
  zsh)
    CONFIG_FILE="$HOME/.zshrc"
    ;;
  bash)
    # Linux 上偏好 .bashrc，macOS 上偏好 .bash_profile
    if [ -f "$HOME/.bashrc" ]; then
      CONFIG_FILE="$HOME/.bashrc"
    else
      CONFIG_FILE="$HOME/.bash_profile"
    fi
    ;;
  fish)
    CONFIG_FILE="$HOME/.config/fish/config.fish"
    ;;
  *)
    echo "❌ Unsupported shell: $SHELL_TYPE"
    echo "   Supported shells: bash, zsh, fish"
    echo "   Usage: /install-claudesidian-command [bash|zsh|fish]"
    exit 1
    ;;
esac

echo "🐚 Installing for: $SHELL_TYPE"
echo "📝 Config file: $CONFIG_FILE"
```

**关键改进**：

- 用 `$SHELL` 检测默认 shell（不用 `$ZSH_VERSION`/`$BASH_VERSION`，那俩检测的是当前会话）
- 支持命令行参数覆盖自动检测
- 显示检测到的 shell 和配置文件，便于核实
- 校验 shell 类型，对不支持的给清楚错误

## 安装步骤

1. **检测 shell**：有参数就用参数，否则从 `$SHELL` 自动检测
2. **拿 vault 路径**：用 `pwd` 取当前目录
3. **转义路径**：正确转义引号和特殊字符确保 shell 安全
   ```bash
   # 先转义反斜杠（这样后面加的反斜杠不会被双重转义）
   ESCAPED_PATH="${VAULT_PATH//\\/\\\\}"
   # 再转义双引号
   ESCAPED_PATH="${ESCAPED_PATH//\"/\\\"}"
   ```
4. **检查是否已装**：在配置文件里搜已有的 `claudesidian` alias / function
   ```bash
   # 检查已有 alias / function
   if grep -q "alias claudesidian\|function claudesidian" "$CONFIG_FILE"; then
     echo "⚠️  Found existing claudesidian command:"
     grep -A 3 "claudesidian" "$CONFIG_FILE"
     echo ""
     read -p "Replace it? (yes/no): " replace_answer
     if [[ ! "$replace_answer" =~ ^[Yy] ]]; then
       echo "Installation cancelled. Existing command preserved."
       exit 0
     fi
     # 标记为替换（添加新的之前会移除旧的）
     REPLACING=true
   fi
   ```
5. **拿用户确认**：显示要加的内容并最终确认
6. **创建备份**：仅在确定要修改时
   ```bash
   # 带时间戳备份
   BACKUP_FILE="$CONFIG_FILE.backup-$(date +%Y%m%d-%H%M%S)"
   cp "$CONFIG_FILE" "$BACKUP_FILE"
   echo "💾 Backup created: $BACKUP_FILE"
   ```
7. **构建安全的 alias / function 命令**：用步骤 3 转义后的路径
   ```bash
   # 关键：在命令里用 $ESCAPED_PATH（不是原始 $VAULT_PATH）
   if [ "$SHELL_TYPE" = "fish" ]; then
     # Fish 用 function 语法，不是 alias
     COMMAND_TEXT="function claudesidian
    cd \"$ESCAPED_PATH\" && (claude --resume 2>/dev/null; or claude)
    cd -
   end"
   else
     # Bash / Zsh 用 alias 语法
     # 重要：$ESCAPED_PATH 外用双引号保留转义
     COMMAND_TEXT="alias claudesidian='(cd \"$ESCAPED_PATH\" && (claude --resume 2>/dev/null || claude))'"
   fi
   ```
8. **替换时移除旧命令**：
   ```bash
   if [ "$REPLACING" = true ]; then
     # Bash / Zsh: alias 是单行 — 只删那行
     sed -i.tmp '/^alias claudesidian/d' "$CONFIG_FILE"
     # Fish: function 跨多行 — 从 `function claudesidian` 删到对应的 `end`
     sed -i.tmp '/^function claudesidian/,/^end$/d' "$CONFIG_FILE"
     rm -f "$CONFIG_FILE.tmp"
   fi
   ```

   **为什么两个独立的 sed**：合并的范围 `/alias claudesidian\|function claudesidian/,/^end$/d` 在 alias 情况下会一直吃行直到找到下一个 `^end$`（或 EOF），可能炸掉下方无关配置。alias 用单行删、function 用范围删 — 绝不合并。
9. **把命令加到配置文件**：用转义后的命令文本追加
   ```bash
   echo "$COMMAND_TEXT" >> "$CONFIG_FILE"
   ```
10. **显示成功消息**：附上重新加载 shell 的指示

## 输出示例

**Bash / Zsh 例子（路径含空格演示转义）**：

```
🔧 Installing claudesidian command...

📁 Vault path: /home/user/My Obsidian Vault
🐚 Shell detected: zsh
📝 Config file: /home/user/.zshrc

💾 Backup created: /home/user/.zshrc.backup-20250107-143025

✅ Installed! Added to /home/user/.zshrc:
   alias claudesidian='(cd "/home/user/My Obsidian Vault" && (claude --resume 2>/dev/null || claude))'

🔄 To activate, run:
   source ~/.zshrc

   Or start a new terminal session.

✨ Test it: Type 'claudesidian' from any directory!
```

**Fish Shell 例子**：

```
🔧 Installing claudesidian command...

📁 Vault path: /home/user/My Obsidian Vault
🐚 Shell detected: fish
📝 Config file: /home/user/.config/fish/config.fish

💾 Backup created: /home/user/.config/fish/config.fish.backup-20250107-143025

✅ Installed! Added to /home/user/.config/fish/config.fish:
   function claudesidian
    cd "/home/user/My Obsidian Vault" && (claude --resume 2>/dev/null; or claude)
    cd -
end

🔄 To activate, run:
   source ~/.config/fish/config.fish

   Or start a new terminal session.

✨ Test it: Type 'claudesidian' from any directory!
```

## 处理特殊字符

实现正确处理含以下字符的路径：

- 空格：`/Users/noah/My Vault`
- 引号：`/Users/noah/vault's backup`
- 需要转义的特殊字符

路径加双引号，内部引号 / 反斜杠会转义。

## Fish Shell 支持

Fish 语法和 Bash / Zsh 不同：

**Bash / Zsh（alias）**：

```bash
alias claudesidian='(cd "/path" && command)'
```

**Fish（function）**：

```fish
function claudesidian
    cd "/path" && (command; or fallback)
    cd -
end
```

关键区别：

- Fish 复杂命令用 `function` 关键字而不是 `alias`
- Fish 用 `; or` 而不是 `||` 做回退逻辑
- Fish 用 `cd -` 回到前一目录（而不是 subshell）
- 多行 function 定义，而不是单行 alias

安装时自动检测 Fish 并用正确语法。

## 安全考量

这个命令会改你的 shell 配置文件（敏感操作）。安全措施：

- **改动前你能看到具体加什么**
- **改动前自动建带时间戳的备份**
- **vault 路径正确转义**防注入
- **只改 claudesidian 命令** — 配置里其他东西不动
- **替换已有命令前会问**

出问题就从 `$CONFIG_FILE.backup-YYYYMMDD-HHMMSS` 恢复。

## 注意事项

- 命令用 subshell `()`（Fish 用 `cd -`）保证执行完回到原目录
- 自动尝试恢复已存会话，否则起新会话
- alias / function 已存在时问用户要不要替换
- 改配置前总是显示要加什么
- **配置改动前永远建带时间戳备份**（格式：`YYYYMMDD-HHMMSS`）
- 备份永久保留 — 用户可手动清旧的
- 显示备份位置，方便用户必要时恢复

## 用法示例

为默认 shell 安装（自动检测）：

```
/install-claudesidian-command
```

为指定 shell 安装（覆盖自动检测）：

```
/install-claudesidian-command zsh
/install-claudesidian-command bash
/install-claudesidian-command fish
```

**什么时候指定 shell**：

- 你用多个 shell，要装到某一个
- 自动检测选错了
- 你在给别人装

## 工作原理

**Bash / Zsh（alias + subshell）**：

```bash
alias claudesidian='(cd "/path/to/vault" && (claude --resume 2>/dev/null || claude))'
```

1. `(cd "/path/to/vault" && ...)` — Subshell 临时切目录（路径双引号保护）
2. `claude --resume 2>/dev/null` — 尝试恢复已存会话，错误抑制
3. `|| claude` — 恢复失败（无会话）就起新的
4. Claude 退出后 subshell 关闭，自动回到原目录

**Fish（function + cd -）**：

```fish
function claudesidian
    cd "/path/to/vault" && (claude --resume 2>/dev/null; or claude)
    cd -
end
```

1. `cd "/path/to/vault"` — 切到 vault 目录（路径双引号保护）
2. `claude --resume 2>/dev/null` — 尝试恢复已存会话，错误抑制
3. `; or claude` — 恢复失败就起新的（Fish 语法）
4. `cd -` — Claude 退出后回到前一目录
