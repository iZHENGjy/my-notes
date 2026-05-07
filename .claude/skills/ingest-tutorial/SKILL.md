---
name: ingest-tutorial
description: 解一份 tutorial / 习题集,产出教学质量的解答(公式速查 + 量级估算 + 推导步骤 + 易错 + 知识盲区)放到 01_Projects/<CODE>_中文名/T##.md。Triggers - "解这份 tutorial" / "做这份习题" / "solve these problems" / "帮我做练习"(不是要交的作业)。
---

# Skill: ingest-tutorial

## Role

产出**教学质量的解答**,不只是答案。一份好的 tutorial 笔记做 4 件事:

1. 上来汇总公式 + 数据(公式速查 + 数据表)
2. 解释**为什么**这步这么做 + 量级估算反向校验(思路与估算)
3. 用 wikilink 引用涉及概念
4. 列知识盲区(用户该复习哪些)

这是学习工具,不是要提交的作业。

## Ethical boundary

本 skill 是**学习辅助**。如果用户说"这是要交的成绩作业",拒绝并改为:
- 复习涉及概念
- 解释方法但不解
- 做相似题

教材习题 / 过往考试 / 练习集都 OK——这些是学习用的。

## When to trigger

- "解这份 tutorial / 做这些练习 / solve these problems"
- "帮我做 T##"(tutorial 编号,不是 assignment)
- 附习题集 / 练习单 / 过往考试

模糊场景(先问):
- "Help with my homework" → 问"这是练习(我可以解)还是提交(我只能审)?"

## Inputs

- 习题集(文本 / 图片 / PDF)
- 课程代码 + tutorial 编号
- 可选:相关 lecture 编号、specific topics

## Outputs

- `01_Projects/<CODE>_中文名/T##_topic_snake.md`
- 知识盲区报告(写在 tutorial 笔记里 + chat 输出)

## Dependencies

启动时读:
- `06_Metadata/Templates/tutorial.md`(笔记模板)
- 相关 lecture 笔记:`01_Projects/<CODE>_中文名/L##_*.md`(按 `week` 字段匹配)

## Workflow

### Step 1: 加载上下文

1. 识别课程 / tutorial 编号
2. 找出相关 lecture 笔记(按 `week` 匹配 `L##_*.md`),Read 加载

### Step 2: 解析问题

把输入拆成编号问题。每题识别:
- 题目原文(verbatim)
- 子题(a / b / c ...)
- 已知 vs 待求
- 隐含假设

### Step 3: 公式速查表

从相关 lecture 的知识块里提取本 tutorial 要用的公式,填 `## 本次公式速查`:

| 公式 | 含义 | 来源 |
|---|---|---|
| $J_A = -D_{AB} \frac{dc_A}{dz}$ | Fick 第一定律 | [[L02_diffusive_mass_transfer]] |

规则:
- 只列实际要用的(不超 ~8 个)
- 每个公式 wikilink 回 lecture 笔记

### Step 4: 数据表

填 `## 本次数据与常数`:

| 符号 | 名称 | 值 | 来源 |
|---|---|---|---|
| $D_{AB}$ | CO₂ 在空气中扩散系数 | $1.6 \times 10^{-5}$ m²/s (298K, 1atm) | 课本 Table 2.1 |

**数据可信度规则**:

| 情况 | 处理 |
|---|---|
| 题目明确给出 | 直接用,标"题目给定" |
| 通用常数(R / g / Avogadro) | 直接用,无需标 |
| 物性数据(Antoine / 临界参数) | 标来源(教材表号 / handbook) |
| AI 不确定 | `> [!warning] 请核实: ...` 标记,指向应查的位置 |

**绝不默默编造物性数据**。宁可留空让用户查,也不给可能错的值。

### Step 5: 逐题解答

按 `06_Metadata/Templates/tutorial.md` 结构:

**> (原题)** — verbatim,英文给的就英文

**涉及知识点**:用 `[[wikilink]]` 引用概念。

**思路与估算**:
1. **思路**(中文):解释**为什么**这么做,不是"我们用 X 方法",是"因为题给的是 Y,Y 适合 X 方法,具体..."
2. **量级估算**:正式算前粗估答案范围。例:"气相 D ~10⁻⁵ m²/s,如果算出 10⁻² 就要查单位。"

**解答**:逐步推导
- 每步一行 motivation + 数学
- LaTeX 全程
- 符号推导先,代数最后
- 最终数值带单位
- **对照估算**:答案出来后跟量级估算比,偏差 >1 个数量级要标注

**最终答案**:单独一行,加粗,带单位。

**易错**:预测学生常错的点。

**变式**:1-2 个 "what if" 扩展。

### Step 6: 答案速查表

所有题解完后,填 `## 答案速查`:

| 题号 | 最终答案 | 涉及概念 |
|---|---|---|
| 1 | $P = 49.9$ kPa | [[状态方程]] |
| 2(a) | $J_A = 3.2 \times 10^{-4}$ mol/(m²·s) | [[Fick 第一定律]] |

### Step 7: 知识盲区报告

填 `## 知识盲区 / Gaps identified`:
- 题目要用但相关 lecture 笔记弱(或没有)的概念
- 题目隐含但用户可能没意识的:"Problem 2 第二部分默认你知道 Clausius-Clapeyron,虽然题目没点名"

### Step 8: 报告

```markdown
## Tutorial solved: <CODE> T##

**Problems**: N solved
**Covers**: [[concept-A]], [[concept-B]]
**公式速查**: N formulas from [[L##_...]]
**数据标记**: N values used, K marked "请核实"

**Gaps identified**:
1. [[concept-X]] — 相关 lecture 笔记弱
2. "Maxwell relations" — 没笔记,考虑 ingest 相关 lecture

**Verification steps for you**:
- 自己做 problem 1 对答案
- 物性数据查课本 appendix
```

## Rules

1. **不只给最终答案** — 必须展示推导步骤
2. **数值答案必须带单位**
3. **不伪造物理/化学数据** — 不确定用 `> [!warning] 请核实` 标记
4. **不声称"已验证"** — 结果是 proposed solution,等用户验
5. **详细计算前必须做量级估算** + 算完反向校验
6. **不给题目难度排序**("trivial" / "简单"等)

## Example

**Good**:
```markdown
## 本次公式速查

| 公式 | 含义 | 来源 |
|---|---|---|
| $PV = nRT$ | 理想气体状态方程 | [[L03_equations_of_state]] |
| $(P + \frac{n^2a}{V^2})(V-nb) = nRT$ | 范德华方程 | [[L03_equations_of_state]] |

## Problem 2

> (原题) Closed rigid vessel 0.1 m³, 2 mol CO₂, 300 K. Calculate
> pressure using (a) ideal gas (b) van der Waals. Discuss deviation.

**涉及知识点**: [[状态方程]], [[范德华方程]]

### 思路与估算

**思路**: 本质是对比理想气体 vs 实际气体修正。关键是理解 `a`(分子间
吸引,降 P)+ `b`(分子体积,升 P)的物理意义。

**量级估算**: P ≈ 2×8.3×300/0.1 ≈ 50 kPa(0.5 atm)。CO₂ 在这条件
偏差应在 1-5% 范围。

### 解答

**(a) 理想气体**:
$$P = \frac{nRT}{V} = \frac{2 \times 8.314 \times 300}{0.1} \approx 49.9 \text{ kPa}$$
和估算一致。

**(b) 范德华**:
$$\left(P + \frac{n^2 a}{V^2}\right)(V - nb) = nRT$$
代入...

**最终答案**: (a) $P_{ideal} = 49.9$ kPa; (b) $P_{vdW} = 48.2$ kPa

偏差 3.4%,和估算 1-5% 吻合。

### 易错

> [!warning]
> - $n^2a/V^2$ 里 $n^2$ 容易写成 $n$
> - 范德华 a 在不同文献单位不同,差 10⁶ 倍要小心
```

**Bad**:无公式表 / 无数据来源 / 无估算 / 直接套数字无解释。

## Failure modes

| 模式 | 触发 | 处理 |
|---|---|---|
| 用户说"这是要交的作业" | 提交场景 | 拒绝直接解,改为复习概念 / 做相似题 |
| 数值答案缺单位 | 题目本身省单位 | 补回 SI 单位,在解答里说"原题省略" |
| 公式表与解答冲突 | 公式不一致 | FAIL,先和用户对齐正确版本 |
| 题号格式混乱 | 1.a vs 1(a) 等 | 统一 `N(a)`,文件头注明"原格式 X" |
| 物性数据不确定 | 题目要用但 AI 不知 | `> [!warning] 请核实` + 指向 handbook 条目,绝不编造 |
