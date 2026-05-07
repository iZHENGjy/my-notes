---
name: upgrade
description: Intelligently upgrade claudesidian with new features while preserving user customizations using AI-powered semantic analysis. Use when the user wants to upgrade claudesidian, pull in upstream changes, or update their installation. / 用 AI 语义分析智能升级 claudesidian，加新功能同时保留用户自定义。当用户要"升级 claudesidian"、"拉上游改动"、"更新安装"时触发。
allowed-tools: [Read, Write, Edit, MultiEdit, Bash, WebFetch, Grep, Glob]
---

# Smart Upgrade

从 GitHub 拉最新 claudesidian 合并到用户的 vault，保护用户自定义。

## 布局假设（先读这个）

claudesidian 把 skill 存在 **`.agents/skills/<name>/SKILL.md`** 作为标准位置。Symlink 在 `.claude/skills/<name>` 和 `.pi/skills/<name>` 指回标准位置。改标准位置，所有 consumer 跟着变。

如果用户在转换前的 claudesidian 上（commands→skills 迁移之前的任何版本），他会有 `.claude/commands/*.md` 而没有 `.agents/skills/`。**那种情况不要跑普通 upgrade** — 看本文件底部的"从老布局迁移"。

如果用户本地仓库有 `.agents/skills/`，正常进行。

## 流程

### 1. 版本检查

- 从 `package.json` 读当前版本
- 取上游版本：
  ```bash
  CURRENT=$(grep '"version"' package.json | head -1 | cut -d'"' -f4)
  LATEST=$(curl -s https://raw.githubusercontent.com/heyitsnoah/claudesidian/main/package.json | grep '"version"' | head -1 | cut -d'"' -f4)
  if [ "$CURRENT" = "$LATEST" ]; then
    echo "✅ Already on $CURRENT"
    exit 0
  fi
  ```
- 检测布局：`test -d .agents/skills && echo NEW || echo LEGACY`。LEGACY 就跳到"从老布局迁移"。

### 2. 备份

```bash
BACKUP_DIR=".backup/upgrade-$(date +%Y-%m-%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"
cp -r .agents .claude .pi .scripts package.json "$BACKUP_DIR/" 2>/dev/null
cp CHANGELOG.md README.md "$BACKUP_DIR/" 2>/dev/null || true
echo "✅ Backup created at $BACKUP_DIR"
```

### 3. 取上游

```bash
git clone --depth=1 --branch=main \
  https://github.com/heyitsnoah/claudesidian.git \
  .tmp/claudesidian-upgrade
```

用户的工作仓库始终断开 origin — 我们只从 `.tmp/` 读。

### 4. 构建升级 checklist

我们关心的系统文件：

- **Skill**：`.agents/skills/<name>/SKILL.md`（标准位置）— 也包括 skill 目录里的资源（辅助 script、references）。
- **Hook**：`.claude/hooks/*.sh`
- **Settings**：`.claude/settings.json`
- **MCP server**：`.claude/mcp-servers/*`
- **Script**：`.scripts/*`
- **Core**：`package.json`、`CHANGELOG.md`、`README.md`

**绝不动**的文件：

- 用户内容文件夹：`00_Inbox/`、`01_Projects/`、`02_Areas/`、`03_Resources/`、`04_Archive/`、`05_Attachments/`、`06_Metadata/`（除非上游改了 `06_Metadata/Templates/`）
- 用户的 `CLAUDE.md`
- `.obsidian/`（用户 Obsidian 设置）
- `vault-config.json`
- `.mcp.json`（含 API key）
- `.git/` 里任何东西

构建 checklist：

```bash
# 双方都有的、只上游有的、只本地有的 skill
diff -qr .agents/skills .tmp/claudesidian-upgrade/.agents/skills 2>/dev/null

# Hook、settings、mcp-servers、script
diff -qr .claude/hooks .tmp/claudesidian-upgrade/.claude/hooks 2>/dev/null
diff -q .claude/settings.json .tmp/claudesidian-upgrade/.claude/settings.json 2>/dev/null
diff -qr .claude/mcp-servers .tmp/claudesidian-upgrade/.claude/mcp-servers 2>/dev/null
diff -qr .scripts .tmp/claudesidian-upgrade/.scripts 2>/dev/null

# 核心文件
diff -q package.json .tmp/claudesidian-upgrade/package.json
diff -q README.md .tmp/claudesidian-upgrade/README.md
diff -q CHANGELOG.md .tmp/claudesidian-upgrade/CHANGELOG.md
```

把发现写到 `.upgrade-checklist.md`，按类别分组，标记 `[ ] pending`、`[x] updated`、`[-] skipped`。

### 5. 逐文件审查

**死规矩 — 不能跳**：

- 改动前永远先给出 diff
- 永远等用户选择，绝不自动选
- 绝不用 `cp -f`。非交互覆盖用 `cat src > dest`
- 每个文件后更新 `.upgrade-checklist.md`

checklist 里每个文件：

1. 给出 `diff -u local upstream`
2. 判断状态：
   - **本地没改、上游没改** → 标 `[-]` 跳过，下一个
   - **上游改了、本地没改** → 问：apply / keep / 看完整 diff
   - **双方都改了（用户自定义）** → 问：保留你的 / 用上游 / 看完整 diff / AI-merge
3. 问用户：
   ```
   File: <path> has updates.
   1. Apply update (take upstream)
   2. Keep your version
   3. View full diff
   4. AI-merge

   Choice (1/2/3/4):
   ```
   **等输入。不要自动选**。
4. 应用选择的动作：
   ```bash
   # 选项 1
   if [ -f ".tmp/claudesidian-upgrade/$path" ]; then
     mkdir -p "$(dirname "$path")"
     cat ".tmp/claudesidian-upgrade/$path" > "$path"
     echo "✅ Updated $path"
   fi
   ```
5. 在 checklist 里标记，继续。

### 6. Symlink 检查（仅 skill）

更新 `.agents/skills/<name>/` 里任何 skill 后，验证 `.claude/skills/<name>` 和 `.pi/skills/<name>` 的 symlink 还指对地方：

```bash
for name in $(ls .agents/skills); do
  for agent in claude pi; do
    link=".${agent}/skills/${name}"
    target="../../.agents/skills/${name}"
    mkdir -p ".${agent}/skills"
    if [ -L "$link" ]; then
      # Symlink 存在 — 验证指向是否正确
      current="$(readlink "$link")"
      if [ "$current" != "$target" ]; then
        rm "$link"
        ln -s "$target" "$link"
        echo "✅ Repaired symlink $link (was → $current)"
      fi
    elif [ -e "$link" ]; then
      # 这个路径上有真实文件或目录 — 不要踩用户数据
      echo "⚠️  $link exists as a real file/dir, not a symlink. Skipping."
      echo "    Manual fix: inspect, back up if needed, then rm and rerun."
    else
      # 啥都没有 — 创建
      ln -s "$target" "$link"
      echo "✅ Created symlink $link"
    fi
  done
done
```

`-L` 测试很关键：损坏 symlink（target 缺失）`-e` 返 false 但 `-L` 返 true，所以只查 `-e` 会试图 `ln -s` 覆盖已存在损坏链接然后失败 "File exists"。反过来，指错方向的 symlink 会通过 `-e`（因为错的 target 还在），然后悄悄保持错。**永远先 `-L`，再用 `readlink` 验 target，再 fallthrough 到 `-e` 看真实文件，啥都没有再创建**。

如果上游引入了全新 skill，这个循环也会建它的 symlink。

### 7. 验证

重跑 step 4 的 diff 命令。剩下的差异应该只是用户明确选择保留的文件（checklist 里标 `[-]` 或 `[x] customized`）。还在 `[ ]` pending 的就是 bug — 报告给用户决定要不要重试。

### 8. 收尾

- 把 `package.json` 版本更新到匹配上游
- `rm -rf .tmp/claudesidian-upgrade`
- 把最终的 `.upgrade-checklist.md` 存到备份目录备查
- 打印总结：updated 数、skipped 数、customized 数

## 冲突解决哲学

skill 上游和本地都改了时，**优先 AI-merge**，而不是 "take upstream"。用户本地改通常代表有意偏好（自己的语气、工作流约定）。上游改通常代表新功能或 bug 修复。**几乎总能两个都留**。

应用前以具体 diff 给出 merge 提案。**不要复述**。

## 更新分类

### Auto-safe（低风险，默认 "apply"）

- 上游引入的全新 skill（增量 — 直接 symlink）
- Hook script 更新（`.claude/hooks/*.sh`）当本地没改时
- `package.json` 里 `dependencies` / `devDependencies` 的依赖 bump（保留用户加的 `scripts` 段）
- `CHANGELOG.md`（总是用上游版本替换）
- `.scripts/` 里的新文件

### 需要审查（总是问）

- 用户碰过的 skill
- `.claude/settings.json`（常有用户加的 hook）
- `package.json` 的 `scripts` 段
- `README.md`
- MCP server 文件

### 绝不动

见 step 4 的"绝不动的文件"列表。

## 错误处理

- **没网** → 优雅失败，建议重试
- **GitHub rate limit** → 等一下重试一次，再失败就报 rate-limit reset 时间
- **AI-merge 时冲突** → 显示两版本，让用户挑
- **`rm -rf .tmp/claudesidian-upgrade` 失败** → 留着，提醒用户

## 回滚

step 2 的备份目录是回滚目标。手动回滚：

```bash
cp -r .backup/upgrade-<timestamp>/.agents .
cp -r .backup/upgrade-<timestamp>/.claude .
cp .backup/upgrade-<timestamp>/package.json .
# 等等
```

不要试图玩选择性回滚 — **整个快照恢复**。

## 从老布局迁移

如果 `test -d .agents/skills` 返 false，用户在转换前的 claudesidian 上。他们的 skill 在 `.claude/commands/*.md`。**普通 upgrade 不会工作** — diff 会显示每个命令"本地删了"、每个 skill"上游加了"。

迁移路径：

1. **停下来解释**。告诉用户他们的布局早于 commands→skills 转换，给出迁移建议。
2. **先备份**（上面 step 2，但要包括 `.claude/commands/`）。
3. **把命令文件移到 skill 目录**：
   ```bash
   mkdir -p .agents/skills
   for f in .claude/commands/*.md; do
     [ -f "$f" ] || continue
     name=$(basename "$f" .md)
     [ "$name" = "README" ] && continue
     mkdir -p ".agents/skills/$name"
     mv "$f" ".agents/skills/$name/SKILL.md"
   done
   ```
4. **加 `name`/`description` frontmatter** 给缺的文件。用 `add-frontmatter` skill。
5. **建 symlink** for `.claude/skills/` 和 `.pi/skills/`（见 step 6）。
6. **校验 frontmatter**。用本仓库 `skill-creator` skill 自带的 `quick_validate.py`：
   ```bash
   for d in .agents/skills/*/; do
     uv run --with pyyaml python \
       .agents/skills/skill-creator/scripts/quick_validate.py "$d"
   done
   ```
   这强制完整 skill schema（name、description、allowed-tools、compatibility、license、metadata），多缺必需 key 都拒绝。

   如果 `uv` 没装或 `skill-creator` 不知怎么没了，回退到这个最小内联检查（只验 `name` 和 `description`）：
   ```bash
   for f in .agents/skills/*/SKILL.md; do
     dir=$(basename "$(dirname "$f")")
     awk '
       BEGIN { in_fm=0; has_name=0; has_desc=0 }
       /^---$/ { in_fm++; next }
       in_fm==1 && /^name:/ { has_name=1 }
       in_fm==1 && /^description:/ { has_desc=1 }
       END {
         if (!has_name) print "  ✗ missing name"
         if (!has_desc) print "  ✗ missing description"
         if (has_name && has_desc) print "  ✓ ok"
       }
     ' "$f" | sed "s|^|  $dir: |"
   done
   ```
7. **然后跑普通 upgrade 流程** 拉额外的上游改动。

迁移是一次性的。成功跑过后，未来 upgrade 走正常流程。

## 选择性升级

用户可能会要的常见子集：

- "只更 skill" → 只 diff `.agents/skills/`
- "只更依赖" → 只更 `package.json` deps
- "只更 hook" → 只 diff `.claude/hooks/`

把这些当成 checklist 的过滤器。审查 / 确认规则一样。
