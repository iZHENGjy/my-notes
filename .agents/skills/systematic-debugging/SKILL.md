---
name: systematic-debugging
description: ALWAYS use before attempting any fix. Never jump to solutions - investigate root cause first. Use when encountering any technical issue, bug, test failure, or unexpected behavior. / 任何修复尝试前必用。绝不跳到解决方案 — 先查根因。任何技术问题、bug、测试失败、意外行为时触发。
---

# Systematic Debugging（系统化调试）

## 概述

随便瞎修浪费时间还引新 bug。临时补丁掩盖底层问题。

**核心原则**：尝试修复**前**永远先找根因。修症状就是失败。

**违反流程的字面意思就是违反调试的精神**。

## 铁律

```
没找到根因，不许修
```

没完成 Phase 1，你不能提修复方案。

## 什么时候用

任何技术问题：

- 测试失败
- 生产 bug
- 意外行为
- 性能问题
- 构建失败
- 集成问题

**特别要用的场景**：

- 时间紧（紧急时容易想猜）
- "再修一下就好了" 看起来很显然
- 已经试过多个修复
- 之前的修复没用
- 你没完全理解问题

**不要因为这些跳过**：

- 问题看起来简单（简单 bug 也有根因）
- 你赶时间（赶时间保证返工）
- 老板要现在就修（系统化比折腾快）

## 四个阶段

每个阶段必须完成才能进下一个。

### Phase 1：根因调查

**尝试任何修复前**：

1. **仔细读错误信息**
   - 别跳过错误或警告
   - 它们经常包含确切的解决方案
   - 完整读 stack trace
   - 注意行号、文件路径、错误代码

2. **稳定复现**
   - 你能可靠地触发它吗？
   - 确切步骤是什么？
   - 每次都发生吗？
   - 不能复现 → 多收数据，别猜

3. **检查最近改动**
   - 改了什么可能引起这个？
   - Git diff、最近 commit
   - 新依赖、配置变更
   - 环境差异

4. **多组件系统里收证据**

   **当系统有多个组件时**（CI → build → signing，API → service → database）：

   **提修复前，加诊断埋点**：

   ```
   每个组件边界：
     - 记录数据进入组件
     - 记录数据离开组件
     - 验证环境 / 配置传递
     - 检查每层的状态

   先跑一次收证据看哪里坏
   然后分析证据找出失败组件
   然后调查那个具体组件
   ```

   **例子（多层系统）**：

   ```bash
   # Layer 1: Workflow
   echo "=== Secrets available in workflow: ==="
   echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

   # Layer 2: Build script
   echo "=== Env vars in build script: ==="
   env | grep IDENTITY || echo "IDENTITY not in environment"

   # Layer 3: Signing script
   echo "=== Keychain state: ==="
   security list-keychains
   security find-identity -v

   # Layer 4: Actual signing
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **这能揭示**：哪一层失败（secrets → workflow ✓，workflow → build ✗）

5. **追踪数据流**

   **错误深在调用栈里时**：

   **快速版**：
   - 坏值从哪起源？
   - 谁用坏值调的这个？
   - 一直追到源头
   - **修源头，不修症状**

### Phase 2：模式分析

**修之前先找模式**：

1. **找可工作的例子**
   - 同代码库里找类似能跑的代码
   - 类似但工作的是什么？

2. **对比参考**
   - 实现某模式时，**完整**读参考实现
   - 别浏览 — 每行都读
   - 应用前完全理解模式

3. **找差异**
   - 工作的和坏的之间什么不同？
   - 列出所有差异，再小也列
   - 别假设"那不可能有影响"

4. **理解依赖**
   - 这需要什么其他组件？
   - 什么设置、配置、环境？
   - 它做了什么假设？

### Phase 3：假设和测试

**科学方法**：

1. **形成单一假设**
   - 清楚说："我认为 X 是根因，因为 Y"
   - 写下来
   - 具体，不模糊

2. **最小测试**
   - 做**最小**可能的改动测假设
   - 一次一个变量
   - **不要**一次修多件事

3. **继续前先验证**
   - 起作用了？是 → Phase 4
   - 没起作用？形成**新**假设
   - **别**在上面叠更多修复

4. **不知道时**
   - 说 "I don't understand X"
   - 别假装懂
   - 求助
   - 多研究

### Phase 4：实现

**修根因，不修症状**：

1. **创建失败测试用例**
   - 最简复现
   - 能自动化就自动化
   - 没框架就一次性脚本
   - 修之前**必须**有

2. **实现单一修复**
   - 针对识别出的根因
   - **一次一个改动**
   - 不"顺便"加改进
   - 不打包重构

3. **验证修复**
   - 测试现在通过？
   - 别的测试没坏？
   - 问题真解决了？

4. **如果修复不工作**
   - **停**
   - 数：试过几次修复？
   - 少于 3 次：回 Phase 1，用新信息重新分析
   - **大于等于 3 次：停下来质疑架构（下面 step 5）**
   - **不要**没架构讨论就尝试第 4 次修复

5. **3 次以上失败：质疑架构**

   **架构问题的模式**：
   - 每次修复揭示新的共享状态 / 耦合 / 不同地方的问题
   - 修复要"大重构"才能实现
   - 每次修复在别处制造新症状

   **停下来质疑根本**：
   - 这个模式根本上 sound 吗？
   - 我们是在"纯惯性坚持"吗？
   - 应该重构架构 vs 继续修症状？

   **试更多修复前先和用户讨论**

   这不是失败的假设 — 这是**错的架构**。

## 红旗 — 停下来按流程走

如果你抓到自己想：

- "先临时修，回头查"
- "随便试改 X 看好不好"
- "加多个改动，跑测试"
- "跳过测试，我手动验证"
- "可能是 X，让我修那个"
- "我没完全懂但这可能管用"
- "模式说 X 但我会改改用"
- "这是主要问题：[没调查就列修复]"
- 在追数据流前就提解决方案
- **"再试一个修复"（已经试过 2+ 次时）**
- **每次修复在别处揭示新问题**

**所有这些都意味着：停。回 Phase 1**。

**3 次以上修复失败**：质疑架构（见 Phase 4.5）

## 常见的合理化借口

| 借口                                       | 现实                                                                 |
| -------------------------------------------- | ----------------------------------------------------------------------- |
| "问题简单，不需要流程"        | 简单问题也有根因。流程对简单 bug 也快。    |
| "紧急，没时间走流程"             | 系统化调试**比**猜测折腾**快**。          |
| "先试这个，再调查"      | 第一次修复定模式。从一开始就做对。                 |
| "确认修复后再写测试" | 没测试的修复站不住。先写测试证明。                       |
| "一次修多个省时间"          | 不能隔离哪个起作用。引新 bug。                             |
| "参考太长，我改改用就好" | 部分理解保证 bug。**完整**读。              |
| "我看到问题了，让我修"           | 看到症状 ≠ 理解根因。                             |
| "再试一个修复"（2+ 次失败后）   | 3+ 次失败 = 架构问题。质疑模式，别再修。 |

## 速查

| 阶段                 | 关键活动                                         | 成功标准            |
| --------------------- | ------------------------------------------------------ | --------------------------- |
| **1. 根因**     | 读错误、复现、查改动、收证据 | 理解**是什么**和**为什么**     |
| **2. 模式**        | 找工作例子、对比                         | 识别差异        |
| **3. 假设**     | 形成理论、最小测试                            | 确认或新假设 |
| **4. 实现** | 建测试、修、验证                               | bug 解决，测试通过    |

## 技术：根因追踪

bug 出现在调用栈深处时，反向追踪找原始触发点。

### 追踪流程

1. **观察症状**

   ```
   Error: git init failed in /Users/jesse/project/packages/core
   ```

2. **找直接原因** — 什么代码直接导致？

   ```typescript
   await execFileAsync('git', ['init'], { cwd: projectDir })
   ```

3. **问：谁调的？**

   ```typescript
   WorktreeManager.createSessionWorktree(projectDir, sessionId)
     → 被 Session.initializeWorkspace() 调
     → 被 Session.create() 调
     → 被 Project.create() 的测试调
   ```

4. **继续往上追** — 传了什么值？
   - `projectDir = ''`（空字符串！）
   - 空字符串作 `cwd` 解析为 `process.cwd()`

5. **找原始触发** — 空字符串从哪来？
   ```typescript
   const context = setupCoreTest() // 返 { tempDir: '' }
   Project.create('name', context.tempDir) // 在 beforeEach 之前访问！
   ```

### 加 stack trace

不能手动追时，加埋点：

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack
  console.error('DEBUG git init:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  })
  await execFileAsync('git', ['init'], { cwd: directory })
}
```

**贴士**：

- 测试里用 `console.error()`（logger 可能被压制）
- 在危险操作**之前**记，不是失败之后
- 含上下文：directory、cwd、环境变量
- `new Error().stack` 显示完整调用链

### 找哪个测试污染

测试期间出现某东西但不知道哪个测试时，二分查找：

```bash
# 一个一个跑测试，第一个污染就停
for f in src/**/*.test.ts; do
  npm test "$f" && [ -d .git ] && echo "POLLUTER: $f" && break
done
```

**绝不只在错误出现的地方修**。回溯找原始触发。

## 技术：纵深防御校验

找到根因后，在数据经过的**每一层**校验。让 bug 在结构上不可能发生。

### 为什么多层

- 单点校验："我们修了 bug"
- 多层："我们让 bug 不可能"

不同层抓不同情况：

- 入口校验抓大多 bug
- 业务逻辑抓边缘情况
- 环境守卫防止上下文特定危险
- 调试日志在其他层失败时帮忙

### 四层

**Layer 1：入口校验** — API 边界拒非法输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty')
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`)
  }
}
```

**Layer 2：业务逻辑校验** — 确保数据对操作有意义

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization')
  }
}
```

**Layer 3：环境守卫** — 防止特定上下文里的危险操作

```typescript
async function gitInit(directory: string) {
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory))
    const tmpDir = normalize(resolve(tmpdir()))
    if (!normalized.startsWith(tmpDir)) {
      throw new Error(`Refusing git init outside temp dir during tests`)
    }
  }
}
```

**Layer 4：调试埋点** — 捕获上下文备查

```typescript
async function gitInit(directory: string) {
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack: new Error().stack,
  })
}
```

### 应用纵深防御

找到 bug 时：

1. **追数据流** — 坏值从哪起源？哪里用？
2. **画所有检查点** — 列出数据经过的每个点
3. **每层加校验** — 入口、业务、环境、调试
4. **测每一层** — 试着绕过 layer 1，验证 layer 2 抓到

**别只在一个校验点停**。每层都加检查。

## 真实影响

来自调试会话：

- 系统化方法：15-30 分钟修
- 随机修：2-3 小时折腾
- 一次修复成功率：95% vs 40%
- 引入新 bug：接近零 vs 常见
