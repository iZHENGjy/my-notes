---
name: pragmatic-review
description: Interactive pragmatic code review focusing on YAGNI and KISS principles. Use when the user asks for a code review, wants to check for over-engineering, or wants a YAGNI/KISS-focused review of changes. / 交互式的务实代码审查，聚焦 YAGNI 和 KISS 原则。当用户要"代码审查"、查"过度工程"、或要 YAGNI/KISS 风格审查时触发。
---

# 务实代码审查：YAGNI & KISS

你要做一次交互式代码审查，重点死磕 **YAGNI**（You Aren't Gonna Need It，你不会需要它的）和 **KISS**（Keep It Simple, Stupid，保持简单）。

## 审查模式

**默认模式**：快速 YAGNI/KISS 聚焦审查

- 扫过度工程、未用抽象、不必要复杂度
- 快速安全和性能检查（OWASP 基础、明显 N+1 查询）
- 自我反思，用证据验证发现

**Deep 模式**（`--deep` flag）：多 pass 全面审查

- Pass 1：安全（OWASP Top 10、输入校验、auth 问题）
- Pass 2：架构（SOLID 原则、关注点分离）
- Pass 3：逻辑（边缘情况、错误处理、正确性）
- Pass 4：性能（算法复杂度、资源泄漏）
- Pass 5：YAGNI/KISS（过度工程、不必要抽象）
- Pass 6：可维护性（可读性、测试、文档）
- 全部跑完做自我反思

什么时候用 `--deep`：

- 安全关键改动（auth、支付、数据处理）
- 核心架构修改
- 多边缘情况的复杂逻辑改动
- 性能敏感的代码路径

什么时候用默认：

- 加功能
- 修 bug
- 重构
- 改文档

**CI 模式**（`--ci` flag）：GitHub Actions 用的非交互模式

- 跳过**所有**交互提示
- 自动选：当前分支 vs base 分支的全部改动
- 有 `$GITHUB_BASE_REF` 环境变量就用
- 一次性输出所有发现（markdown 摘要）

## Step 1：决定审查范围

### 检查当前 Git 状态

先确认在 git 仓库里：

- `test -d .git` 检查 .git 目录

不在 git 仓库就让用户手动指定要审的文件。

在 git 仓库里就收集信息：

#### 当前分支：
跑：`git rev-parse --abbrev-ref HEAD`

#### 默认分支检测：
1. 试：`git rev-parse --verify main`
2. 失败试：`git rev-parse --verify master`
3. 失败试：`git rev-parse --verify develop`

用户在参数里给了 `--base [branch]` 就用那个。

#### 工作区状态：
跑：`git status --short | head -20`

### 给用户选项

**有 `--ci` flag**：跳过所有交互，自动选第 2 项：当前分支 vs base 的所有改动。

没有 `--auto` 或 `--ci` flag 就问用户：

```
📋 CODE REVIEW SCOPE SELECTION
════════════════════════════════

What would you like to review?

1️⃣  Current uncommitted changes
2️⃣  All changes on current branch (compared to [detected default branch])
3️⃣  Specific files or directory
4️⃣  Last N commits
5️⃣  Staged changes only

Please enter your choice (1-5):
```

## Step 2：YAGNI/KISS 分析框架

每个识别出的文件，分析这些模式：

### YAGNI 检测模式

1. **未用的抽象**
   - 只有一个实现的 interface / protocol
   - 只有一个具体子类的抽象基类
   - 总是用同一种类型的泛型

2. **过早灵活**
   - 永远不变的东西做配置
   - 没插件的插件系统
   - 永远开 / 关的 feature flag

3. **过度工程指标**
   - 简单对象的 Factory 类
   - 2-3 个字段对象的 Builder 模式
   - 单 listener 的事件系统

4. **臆测的代码**
   - "TODO: 可能要" 注释
   - "万一" 的注释代码
   - 不可达代码路径
   - 永远不调的方法

5. **GenericButton 反模式**
   - 8+ 可选参数、服务多种用例的组件
   - props 多到用它和从头写一样复杂

6. **过早抽象 — Rule of Three（三次原则）**
   - 第 1、2 次重复就抽象（等到第 3 次！）
   - 参考：Martin Fowler — "Tolerate duplication twice, refactor on the third"

### KISS 违反模式

1. **冗长实现**
   - 能砍掉超过 50% 行数
   - 重复实现标准库
   - 简单字符串就够却用复杂正则

2. **抽象成瘾**
   - 继承 / 包装超过 3 层
   - 每层之间都加 interface

3. **聪明代码**
   - 需要大量注释解释
   - 不必要地用冷僻语言特性
   - 应该 5 行清楚写的写成一行

4. **Catch-Log-Exit 反模式**
   - 抓异常只为打日志和退出
   - 把真错误替换成对出错原因的猜测

   ```typescript
   // 糟糕：把真错误替换成猜的
   try {
     await createNewBranch({ branchName, cwd })
   } catch (error) {
     console.error('Error: Not in a git repository') // 可能是错的！
     process.exit(1)
   }

   // 正确：让它自然抛
   await createNewBranch({ branchName, cwd })
   ```

### 要查的安全模式

哪怕 YAGNI/KISS 审查也要标关键安全问题：

1. **SQL 注入**
   - SQL 查询里字符串拼接
   - 缺参数化查询

2. **认证 / 授权**
   - 硬编码 secret
   - 弱默认：`SECRET = os.getenv('KEY', 'default')`
   - JWT 没过期时间

3. **未校验的外部输入**
   - URL 参数不校验直接用
   - API 响应不做 schema 校验就信

### 要查的性能模式

标明显性能问题：

1. **N+1 查询**
   - 循环里调数据库
   - 缺 eager loading

2. **低效算法**
   - O(n²) 但 O(n) 或 O(n log n) 就够
   - 不必要的嵌套循环

## Step 3：执行分析

**检查 `--deep` flag**：有就用多 pass 深度模式（6 趟）。否则用快速 YAGNI/KISS 模式。

**重要**：只分析本审查范围里实际改动的代码。**不要**报既存问题。

## Step 3.5：自我审查 Pass

**呈现发现前，校验每个问题**：

1. **证据检查**：
   - 我能给出支持这条批评的链接 / 引用吗？
   - 我解释了**为什么**重要吗？

2. **严重度校验**：
   - 这个评级（高 / 中 / 低）准吗？
   - 这个问题真的会引起麻烦吗？

3. **YAGNI 专项**：
   - 标重复时：这是第 3 次以上出现吗？
   - 等以后信息多了再重构行吗？

**通过不了这些检查的，删除或降级。**

## Step 4：交互式审查流程

### 问题严重度前缀

用这些前缀传达优先级：

| 前缀          | 含义                               | 要采取的行动           |
| ------------- | ---------------------------------- | --------------------- |
| `issue:`      | Bug、正确性问题                     | 合并前必须修         |
| `nit:`        | 小改进、风格                         | 可选，不挡合并         |
| `thought:`    | 设计考量                            | 讨论，可推迟         |
| `suggestion:` | 带代码的具体改进                     | 严肃考虑             |

### 交互式逐项审查

每个问题呈现：

```
═══════════════════════════════════════
Issue [current] of [total]
═══════════════════════════════════════

📁 文件: [filename]
📍 行: [start-end]
🏷️  类型: [YAGNI | KISS | 都有]
🎯 严重度: [高 | 中 | 低]

当前代码:
[显示真实代码片段]

发现的问题: [具体描述]

为什么重要: [解释真实成本 / 问题]

建议简化:
[显示更简单的替代代码]

═══════════════════════════════════════

你想怎么做?
1. ✅ 接受 - 加入待修列表
2. ❌ 跳过 - 保留当前代码
3. 💬 讨论 - 标记给团队复审
4. 👀 上下文 - 看更多周围代码
5. ⏹️ 停止 - 在这里结束审查
```

## Step 5：核心审查规则

### **总是**标这些 YAGNI 问题：

1. **只有一个实现的 interface**
2. **未用代码** — 零调用方的函数 / 方法
3. **臆测的数据库字段** — 永远 NULL 的列
4. **过早优化** — 没测就加缓存

### **总是**标这些 KISS 违反：

1. **重新实现标准库**
2. **过多抽象层**
3. **配置压过约定** — 50 行代码配 100 行配置

### **不要**标这些：

1. **必要复杂度** — 错误处理、安全措施
2. **领域复杂度** — 业务规则本来就复杂
3. **团队约定** — 已达成共识的模式

## Step 6：最终总结

```
📝 PRAGMATIC REVIEW COMPLETE
═══════════════════════════════

Review Statistics:
• Files reviewed: [X]
• Lines changed: [Y]

Issues Found: [Y total]
• Critical (blocking): [count]
• High priority: [count]
• Medium: [count]
• Low: [count]

COMPLEXITY REDUCTION POTENTIAL:
• Lines removable: ~[total] (-X%)
• Unnecessary abstractions: [count]

TOP 3 QUICK WINS:
1. [Biggest impact, easiest change]
2. [Second biggest impact]
3. [Third biggest impact]

RECOMMENDATION: [Clear ship/don't ship with reasoning]

═══════════════════════════════
```

## 命令参数参考

- `--auto`：跳过交互提示，用默认（未提交改动）
- `--ci`：CI 模式 — 跳过所有提示，审查分支 vs base
- `--deep`：启用 6-pass 全面审查
- `--branch [name]`：审查指定分支
- `--base [branch]`：和这个 base 分支对比

例：

- `/pragmatic-review` — 交互模式
- `/pragmatic-review --auto` — 自动审查当前改动
- `/pragmatic-review --ci` — GitHub Actions CI 模式
- `/pragmatic-review --deep` — 全面 6-pass 审查

## 核心哲学

不确定时记住：

1. **YAGNI**：功能要付 4 倍代价 — 写、维护、修、机会成本
2. **KISS**：调试比写代码难一倍 — 你写最聪明的代码，按定义你就不够聪明去调它
3. **Rule of Three**：忍两次重复，第三次再重构
4. **务实**：今天发能用的软件，明天再完美它

你的角色是简单性的捍卫者。**每删一行都是胜利**。

## 参考

- Martin Fowler — YAGNI: https://martinfowler.com/bliki/Yagni.html
- KISS principle: https://en.wikipedia.org/wiki/KISS_principle
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- Addy Osmani — "Avoid Large Pull Requests"
- Jeff Atwood — "Curly's Law: Do One Thing"
