---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy. / 创建新 skill、改进已有 skill、测量 skill 表现。当用户要"从零创建 skill"、"编辑 / 优化已有 skill"、"跑 eval 测 skill"、"做 skill benchmark + 方差分析"、或优化 skill 的 description 以获得更好触发准确度时触发。
---

# Skill Creator

一个用于创建新 skill 并迭代改进的 skill。

总体上，创建 skill 的流程像这样：

- 决定你想让 skill 做什么、大概怎么做
- 写一份 skill 草稿
- 创几个测试 prompt，让带有 skill 的 Claude 跑一下
- 帮用户做定性 + 定量评估
  - 跑测试期间在后台进行，先起草定量 eval（如果还没有；有的话可以直接用，或视需要修）。然后给用户解释（已存在就解释已有的）
  - 用 `eval-viewer/generate_review.py` 给用户看结果，让他看定量指标
- 基于用户对结果的评估反馈重写 skill（如果定量基准里有明显缺陷也加入考虑）
- 直到你满意为止
- 扩大测试集，更大规模再试

你用这个 skill 时的工作是：搞清楚用户在哪一步，然后跳进来帮他往下推进。比如，他可能说"我想做一个 X 的 skill"。你可以帮他把意图缩小、写草稿、写测试用例、决定怎么评估、跑所有 prompt、循环。

或者，他已经有了 skill 草稿。这种情况你可以直奔 eval / 迭代环节。

当然你应该灵活 — 如果用户说"我不需要跑一堆 eval，凭感觉来吧"，那就照做。

skill 写好后（顺序也灵活），你还可以跑 description 优化器（我们有专门 script），优化 skill 的触发。

OK？OK。

## 跟用户沟通

skill creator 可能被各种熟悉度的人用 — 编程黑话不熟的也有。你应该知道（怎么会不知道呢，最近才开始的趋势）：Claude 的能力让管道工打开终端，让父母祖父母 google "how to install npm"。但大部分用户大概是相当电脑通的。

所以请注意上下文线索来调整说话方式！默认情况下，给你点感觉：

- "evaluation"（评估）和 "benchmark"（基准）边缘但 OK
- "JSON" 和 "assertion" 要看到用户认得这些的明显信号才能不解释直接用

不确定时简单解释术语 OK，怀疑用户可能不懂时大胆加个简短定义。

---

## 创建 skill

### 抓意图

先理解用户意图。当前对话可能已经包含他想固化的工作流（比如他说"把这个变成 skill"）。这种情况下，先从对话历史抽答案 — 用了什么工具、步骤序列、用户做了哪些纠正、观察到的输入 / 输出格式。用户可能要补缺，进下一步前要他确认。

1. 这个 skill 应该让 Claude 能做什么？
2. 什么时候触发？（用户什么短语 / 上下文）
3. 期望输出格式？
4. 要不要建测试用例验证 skill 工作？输出可客观验证的 skill（文件转换、数据抽取、代码生成、固定流程）适合测试用例。输出主观的（写作风格、艺术）通常不需要。基于 skill 类型建议合适默认值，让用户决定。

### 访谈和研究

主动问边缘情况、输入 / 输出格式、示例文件、成功标准、依赖。在问清楚之前别急着写测试 prompt。

看可用的 MCP — 对研究有帮助的（搜文档、找类似 skill、查最佳实践），有 subagent 就并行研究，否则内联。带着上下文来减少用户负担。

### 写 SKILL.md

基于用户访谈，填这些组件：

- **name**：skill 标识符
- **description**：什么时候触发、做什么。这是主触发机制 — 既要写 skill 做什么**也**要写具体何时使用。所有"何时使用"的信息都放这，不放正文。注意：当前 Claude 倾向于"漏触发" skill — 该用没用。为了对抗这个，请把 skill description 写得稍微"主动"一点。比如，与其写 "How to build a simple fast dashboard to display internal Anthropic data."，可以写 "How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"
- **compatibility**：必需的工具、依赖（可选，很少需要）
- **skill 的其他部分 :)**

### Skill 写作指南

#### 一个 skill 长什么样

```
skill-name/
├── SKILL.md（必需）
│   ├── YAML frontmatter（name、description 必需）
│   └── Markdown 指令
└── 打包资源（可选）
    ├── scripts/    - 用于确定性 / 重复性任务的可执行代码
    ├── references/ - 按需加载到 context 的文档
    └── assets/     - 输出里用的文件（模板、icon、字体）
```

#### 渐进披露（Progressive Disclosure）

skill 用三级加载系统：
1. **元数据**（name + description）— 始终在 context（约 100 词）
2. **SKILL.md 正文** — skill 触发时加载到 context（理想 < 500 行）
3. **打包资源** — 按需（无限制，script 可执行而不加载）

字数大致估计，需要可以更长。

**关键模式**：
- SKILL.md 控制在 500 行以下；接近时加一层层级 + 清楚指示该看哪
- 在 SKILL.md 里清楚引用文件，附"何时读"指引
- 大的参考文件（>300 行）加目录

**领域组织**：skill 支持多领域 / 框架时，按变体组织：
```
cloud-deploy/
├── SKILL.md（工作流 + 选择）
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```
Claude 只读相关 reference 文件。

#### 不意外原则（Lack of Surprise）

不用说也明白：skill 不能含恶意软件、利用代码、或任何危及系统安全的内容。skill 的内容描述给用户时不应让其意图意外。**别**配合创建误导性 skill 或为非授权访问、数据外泄、其他恶意活动服务的 skill。"扮演 XYZ 角色"这种是 OK 的。

#### 写作模式

指令优先用祈使式。

**定义输出格式** — 可以这样：
```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

**例子模式** — 加例子很有用。可以这样格式（如果 Input / Output 在例子里你可能要稍偏离）：
```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### 写作风格

**解释为什么**重要，而不是堆"必须 MUST"的霸道命令。用 theory of mind，让 skill 通用一些，不要太窄绑死具体例子。先写一稿，然后用新眼光看一遍改进。

### 测试用例

写完 skill 草稿，想 2-3 个真实测试 prompt — 真用户实际会说的那种。给用户看：[不一定要用这个原话]"这是几个我想试的测试用例，看着对吗，要不要加？" 然后跑。

把测试用例存到 `evals/evals.json`。**别**急着写 assertion — 只写 prompt。下一步在跑测试时起草 assertion。

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

完整 schema（含 `assertions` 字段，稍后再加）见 `references/schemas.md`。

## 跑测试用例并评估

这一段是连续的 — **不要**中途停。**不要**用 `/skill-test` 或其他测试 skill。

把结果放到 `<skill-name>-workspace/`，作为 skill 目录的兄弟。workspace 里按迭代组织（`iteration-1/`、`iteration-2/` 等），里面每个测试用例一个目录（`eval-0/`、`eval-1/`）。**别**一开始全部建好 — 边走边建。

### Step 1：同一回合 spawn 所有运行（with-skill **和** baseline）

每个测试用例，同一回合 spawn 两个 subagent — 一个带 skill，一个不带。这很重要：**别**先 spawn with-skill 然后回头再做 baseline。一次性都启动，让它们差不多同时结束。

**With-skill 运行**：

```
Execute this task:
- Skill path: <path-to-skill>
- Task: <eval prompt>
- Input files: <eval files if any, or "none">
- Save outputs to: <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Outputs to save: <what the user cares about — e.g., "the .docx file", "the final CSV">
```

**Baseline 运行**（同 prompt，但 baseline 看场景）：
- **创建新 skill**：完全没 skill。同 prompt，不传 skill path，存到 `without_skill/outputs/`。
- **改进已有 skill**：旧版本。改之前先 snapshot（`cp -r <skill-path> <workspace>/skill-snapshot/`），然后让 baseline subagent 指向 snapshot。存到 `old_skill/outputs/`。

每个测试用例写 `eval_metadata.json`（assertion 现在可以为空）。每个 eval 起描述性名字 — 不只是 "eval-0"。目录也用这个名。如果本次迭代用了新或改动的 eval prompt，给每个新 eval 目录建这些文件 — **别**假设它们从前次迭代继承。

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": []
}
```

### Step 2：运行进行时起草 assertion

别只是等运行结束 — 这段时间能干活。给每个测试用例起草定量 assertion，并给用户解释。如果 `evals/evals.json` 里已有 assertion，复审并解释每条查什么。

好的 assertion 是客观可验证的，名字描述性强 — benchmark viewer 里读起来要清楚，扫一眼就懂每条查什么。主观 skill（写作风格、设计质量）更适合定性评估 — **别**强行给需要人判断的东西加 assertion。

起草完更新 `eval_metadata.json` 和 `evals/evals.json`。给用户解释 viewer 里能看到什么 — 定性输出和定量基准。

### Step 3：运行完成时捕获时间数据

每个 subagent 任务完成时，你会收到含 `total_tokens` 和 `duration_ms` 的通知。立刻存到运行目录的 `timing.json`：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

**这是捕获这数据的唯一机会** — 它通过任务通知传过来，别处没保存。**每个通知到达时立即处理**，别试图批量。

### Step 4：评分、聚合、启动 viewer

所有运行完成后：

1. **每个运行评分** — spawn grader subagent（或内联评分），让它读 `agents/grader.md`，对每条 assertion 评估输出。结果存到每个运行目录的 `grading.json`。grading.json 的 expectations 数组**必须**用字段 `text`、`passed`、`evidence`（不是 `name`/`met`/`details` 或其他变体）— viewer 依赖这些精确字段名。**能编程检查的 assertion，写脚本跑而不是肉眼看** — 脚本快、可靠、可跨迭代复用。

2. **聚合到 benchmark** — 在 skill-creator 目录跑聚合脚本：
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
   产出 `benchmark.json` 和 `benchmark.md`，含每个配置的 pass_rate、time、tokens（mean ± stddev + delta）。手动生成 benchmark.json 就看 `references/schemas.md` 的精确 schema。
把每个 with_skill 版本放在对应 baseline 之前。

3. **做一遍 analyst 分析** — 读 benchmark 数据，浮上聚合统计可能隐藏的模式。看 `agents/analyzer.md`（"Analyzing Benchmark Results" 段）找什么 — 比如不论 skill 都通过的 assertion（不区分）、高方差 eval（可能 flaky）、time / token 权衡。

4. **启动 viewer**，附定性输出和定量数据：
   ```bash
   nohup python <skill-creator-path>/eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-skill" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     > /dev/null 2>&1 &
   VIEWER_PID=$!
   ```
   迭代 2 以上还要传 `--previous-workspace <workspace>/iteration-<N-1>`。

   **Cowork / 无界面环境**：如果 `webbrowser.open()` 不可用或环境无显示，用 `--static <output_path>` 写出独立 HTML 文件而不是起服务器。用户点 "Submit All Reviews" 时 feedback 会下载为 `feedback.json`。下载后把它复制到 workspace 目录，下次迭代会取。

注意：请用 generate_review.py 创建 viewer，不需要写自定义 HTML。

5. **告诉用户**类似的话："我已在浏览器打开结果。两个 tab — 'Outputs' 让你过每个测试用例留反馈，'Benchmark' 显示定量对比。看完回来告诉我。"

### viewer 里用户看到什么

"Outputs" tab 一次显示一个测试用例：
- **Prompt**：给的任务
- **Output**：skill 产生的文件，能内联就内联渲染
- **Previous Output**（迭代 2+）：折叠段，显示上次迭代的输出
- **Formal Grades**（如果跑了评分）：折叠段，显示 assertion 通过 / 失败
- **Feedback**：自动保存的文本框
- **Previous Feedback**（迭代 2+）：上次的评论，显示在文本框下方

"Benchmark" tab 显示统计摘要：每个配置的通过率、耗时、token 使用，含逐 eval 拆分和 analyst 观察。

导航靠 prev/next 按钮或方向键。完成时点 "Submit All Reviews" 把所有反馈存到 `feedback.json`。

### Step 5：读反馈

用户告诉你做完时，读 `feedback.json`：

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "the chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

空反馈意思是用户觉得没问题。改进聚焦于用户有具体抱怨的测试用例。

完事关掉 viewer server：

```bash
kill $VIEWER_PID 2>/dev/null
```

---

## 改进 skill

这是循环的核心。你跑了测试用例，用户复审了结果，现在你要基于反馈把 skill 做得更好。

### 怎么思考改进

1. **从反馈泛化**。大局上讲，我们在创建可以被用百万次（也许真的，也许更多谁知道）跨多种 prompt 的 skill。这里你和用户对几个例子反复迭代是因为它能让节奏快。用户对这些例子门儿清，能快速评估新输出。但**如果你和用户共同开发的 skill 只对这些例子好用，那它没用**。比起加细碎过拟合的改动、或压抑性的"必须 MUST"，遇到顽固问题，可以试试岔开路径用不同隐喻、推荐不同工作模式。试错代价相对低，可能就 land 在好东西上。

2. **prompt 保持精瘦**。删不出力的东西。**读 transcript，不只是看最终输出** — 如果看起来 skill 让模型浪费时间做没产出的事，就把那些部分删掉看会怎样。

3. **解释 why**。**用力解释**你要求模型做的每件事**为什么**。今天的 LLM **聪明**。它们 theory of mind 好，给个好的 harness 就能超越死板指令真的搞定事情。哪怕用户反馈简洁或挫败，努力理解任务、为什么用户写他写的、他实际写了什么，把这种理解传递到指令里。如果你发现自己用 ALWAYS 或 NEVER 全大写，或用超死板结构，**这是黄旗** — 可能的话重新表述并解释道理，让模型理解为什么你要的事情重要。这是更人性化、更强大、更有效的方法。

4. **找跨测试用例的重复工作**。读测试运行的 transcript，注意 subagent 是不是都独立写了类似的 helper script，或对某事用了同样的多步方法。如果 3 个测试用例 subagent 都写了 `create_docx.py` 或 `build_chart.py`，这是 skill 该打包那个 script 的强信号。**写一次，放 `scripts/`，让 skill 用它**。这能省掉每次未来调用重新发明轮子。

这个任务挺重要（我们要创造每年数十亿的经济价值！）你的思考时间不是瓶颈，**慢慢来真的琢磨**。建议写一稿后用新眼光看再改进。**真的尽力进入用户脑袋**理解他想要什么、需要什么。

### 迭代循环

改进 skill 后：

1. 把改进应用到 skill
2. 把所有测试用例跑到新 `iteration-<N+1>/` 目录，包括 baseline 运行。创建新 skill 时 baseline 永远是 `without_skill`（没 skill），跨迭代不变。改进已有 skill 时，自己判断 baseline 用什么合理：用户原本的版本，或前次迭代。
3. 启动 reviewer，`--previous-workspace` 指向前次迭代
4. 等用户审完告诉你做完了
5. 读新反馈，再改进，循环

继续直到：
- 用户说他满意
- 反馈全空（看起来都好）
- 你不在做有意义的进展

---

## 进阶：盲对比

需要更严格对比两版 skill 的场景（比如用户问"新版真比旧版好吗"），有盲对比系统。详见 `agents/comparator.md` 和 `agents/analyzer.md`。基本想法：把两个输出给独立 agent，不告诉它哪个是哪个，让它判质量，然后分析为什么胜者胜。

这是可选的，需要 subagent，多数用户不需要。**人审循环通常足够**。

---

## Description 优化

SKILL.md frontmatter 里的 description 字段是决定 Claude 是否调起 skill 的主机制。创建或改进 skill 后，提议优化 description 提升触发准确度。

### Step 1：生成触发 eval 查询

创建 20 条 eval 查询 — 应触发和不应触发混合。存为 JSON：

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

查询必须真实，是 Claude Code 或 Claude.ai 用户**实际会输入**的东西。**不是抽象请求**，而是具体、有相当细节的请求。比如文件路径、用户工作 / 处境的个人上下文、列名和值、公司名、URL。一点背景故事。有些可能小写或含缩写、错字、口语。用混合不同长度，**聚焦边缘情况**而不是过于明确（用户会有机会确认）。

差例：`"Format this data"`、`"Extract text from PDF"`、`"Create a chart"`

好例：`"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"`

**should-trigger 查询**（8-10 条）想覆盖度。同意图不同表述 — 一些正式，一些随意。包含用户没明确点名 skill 或文件类型但显然需要的情况。塞几个少见用例和这个 skill 与另一个竞争但应胜出的情况。

**should-not-trigger 查询**（8-10 条）最有价值的是**near-miss** — 和 skill 共享关键词或概念但实际需要别的。想想相邻领域、模糊措辞（朴素关键词匹配会触发但不该）、查询接触 skill 做的事但在另一个工具更合适的上下文。

要避免的关键：**别**让 should-not-trigger 查询明显不相关。"写斐波那契函数"作为 PDF skill 的负样本太容易了 — 啥都没测。负例应该真的有难度。

### Step 2：和用户复审

用 HTML 模板把 eval 集给用户审：

1. 从 `assets/eval_review.html` 读模板
2. 替换占位符：
   - `__EVAL_DATA_PLACEHOLDER__` → eval 项的 JSON 数组（外面别加引号 — 它是 JS 变量赋值）
   - `__SKILL_NAME_PLACEHOLDER__` → skill 的 name
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → skill 当前 description
3. 写到临时文件（如 `/tmp/eval_review_<skill-name>.html`）打开：`open /tmp/eval_review_<skill-name>.html`
4. 用户能编辑查询、切换 should-trigger、增删条目，点 "Export Eval Set"
5. 文件下载到 `~/Downloads/eval_set.json` — 看 Downloads 文件夹找最新版（如果有多个比如 `eval_set (1).json`）

这步重要 — **差的 eval 查询导致差的 description**。

### Step 3：跑优化循环

告诉用户："这要花点时间 — 我会在后台跑优化循环并定期检查。"

把 eval 集存到 workspace，然后后台跑：

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id-powering-this-session> \
  --max-iterations 5 \
  --verbose
```

用你 system prompt 里的 model ID（驱动当前会话的那个），让触发测试匹配用户实际体验。

跑的时候定期 tail 输出，给用户更新当前迭代和分数情况。

它会自动处理整个优化循环。把 eval 集分成 60% 训练 + 40% 留出测试，评估当前 description（每个查询跑 3 次拿可靠触发率），然后调 Claude 基于失败提改进。它在训练和测试集上重新评估每个新 description，最多迭代 5 次。完成时在浏览器打开 HTML 报告显示每次迭代结果，返回含 `best_description` 的 JSON — 按测试分而非训练分选，避免过拟合。

### skill 触发怎么工作

理解触发机制有助于设计更好的 eval 查询。skill 出现在 Claude 的 `available_skills` 列表里，带 name + description，Claude 基于 description 决定是否查阅 skill。要知道的关键：**Claude 只对自己难处理的任务查阅 skill** — 简单单步查询如"读这个 PDF"可能不触发 skill，哪怕 description 完美匹配，因为 Claude 用基础工具就能直接处理。复杂、多步、专门的查询当 description 匹配时可靠触发。

意思是你的 eval 查询要**充分**到 Claude 真能受益于查阅 skill。简单查询如"读文件 X"是差测试用例 — 不论 description 质量都不会触发。

### Step 4：应用结果

从 JSON 输出取 `best_description`，更新 skill 的 SKILL.md frontmatter。给用户看 before/after 并报告分数。

---

### 打包并呈现（仅当 `present_files` 工具可用）

检查你是否有 `present_files` 工具。没有就跳过这步。有就打包 skill 并把 .skill 文件呈给用户：

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

打包后，把生成的 `.skill` 文件路径告诉用户，让他安装。

---

## Claude.ai 专门指引

Claude.ai 里核心工作流一样（草稿 → 测 → 审 → 改 → 循环），但因为 Claude.ai 没 subagent，一些机制变了。要适配的：

**跑测试用例**：没 subagent 就没并行。每个测试用例，读 skill 的 SKILL.md，按指令亲自完成测试 prompt。一个一个做。这没独立 subagent 严格（你写 skill 又跑它，有完整 context），但作为完整性检查有用 — 而且人审步骤补偿了。**跳过 baseline 运行** — 直接用 skill 完成任务。

**复审结果**：不能开浏览器（如 Claude.ai 的 VM 无显示，或在远程服务器），完全跳过浏览器 reviewer。改在对话里直接呈现结果。每个测试用例显示 prompt 和输出。如果输出是用户要看的文件（如 .docx 或 .xlsx），存到文件系统并告诉他位置以下载和检查。**内联问反馈**："这看起来怎样？要改什么？"

**Benchmark**：跳过定量基准 — 它依赖 baseline 对比，没 subagent 没意义。聚焦用户的定性反馈。

**迭代循环**：和之前一样 — 改进 skill、重跑测试用例、问反馈 — 只是中间没有浏览器 reviewer。如果有文件系统，仍可把结果组织到迭代目录。

**Description 优化**：本节需要 `claude` CLI 工具（具体是 `claude -p`），仅 Claude Code 有。Claude.ai 上跳过。

**盲对比**：需要 subagent。跳过。

**打包**：`package_skill.py` 在任何有 Python 和文件系统的地方都能跑。Claude.ai 上能跑，用户能下生成的 `.skill` 文件。

**更新已有 skill**：用户可能要你更新已有 skill，不是新建。这种情况：
- **保留原始 name**。注意 skill 目录名和 `name` frontmatter 字段 — 不动。例：装好的 skill 是 `research-helper`，输出 `research-helper.skill`（不是 `research-helper-v2`）。
- **改之前先复制到可写位置**。装好的 skill 路径可能只读。复制到 `/tmp/skill-name/`，在那改，从副本打包。
- **手动打包就先在 `/tmp/` 暂存**，再复制到输出目录 — 直接写可能因权限失败。

---

## Cowork 专门指引

在 Cowork 里要知道的主要事情：

- 你有 subagent，所以主工作流（并行 spawn 测试用例、跑 baseline、评分等）都能用。（不过遇到严重 timeout 问题时，串行跑测试 prompt 也 OK。）
- 没浏览器或显示，所以生成 eval viewer 时用 `--static <output_path>` 写独立 HTML 文件而不是起服务器。然后给用户一个能点开的链接，在浏览器打开 HTML。
- 不知道为什么 Cowork 设置似乎让 Claude 在跑完测试后不愿生成 eval viewer，所以再强调一次：不论 Cowork 还是 Claude Code，**跑完测试后总是要先生成 eval viewer 让人看例子**，再自己改 skill 试图修正，用 `generate_review.py`（不要写自己的精装 HTML 代码）。先抱歉一下我要全大写：在自己评估输入**之前**生成 EVAL VIEWER。要尽快把它放到人面前！
- 反馈方式不同：没运行 server，viewer 的 "Submit All Reviews" 按钮会下载 `feedback.json` 文件。然后从那读（可能要先请求访问）。
- 打包能用 — `package_skill.py` 只要 Python 和文件系统。
- description 优化（`run_loop.py` / `run_eval.py`）在 Cowork 应该没问题，因为它通过 subprocess 用 `claude -p`，不用浏览器，但请等 skill 完全做完用户也认可后再做。
- **更新已有 skill**：用户可能要你更新已有 skill，不是新建。按上面 claude.ai 段的更新指引。

---

## 参考文件

agents/ 目录含专门 subagent 的指令。需要 spawn 相关 subagent 时读它们。

- `agents/grader.md` — 怎么评估 assertion vs 输出
- `agents/comparator.md` — 怎么做盲 A/B 对比
- `agents/analyzer.md` — 怎么分析为什么一个版本胜另一个

references/ 目录含额外文档：
- `references/schemas.md` — evals.json、grading.json 等的 JSON 结构

---

再重复一遍核心循环以强调：

- 搞清楚 skill 是关于什么的
- 起草或编辑 skill
- 在测试 prompt 上跑带 skill 的 Claude
- 和用户一起评估输出：
  - 创建 benchmark.json 跑 `eval-viewer/generate_review.py` 帮用户复审
  - 跑定量 eval
- 直到你和用户都满意
- 打包最终 skill 给用户

请把步骤加到你的 TodoList（如果你有的话）确保不忘。如果在 Cowork，请特别把 "Create evals JSON and run `eval-viewer/generate_review.py` so human can review test cases" 放到 TodoList 里确保发生。

祝好运！
