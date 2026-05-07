---
name: ingest-lecture
description: 把 PPT/PDF 课程材料整理成一份知识块组织的笔记,放到 01_Projects/<CODE>_中文名/L##.md。Triggers - "整理这节课" / "处理这份 PPT" / "ingest lecture" / "process slides" / "把这份 ppt 变成笔记"。
---

# Skill: ingest-lecture

## Role

把一份 lecture 材料(PPT / PDF / 截图 / 文本)转成**一份**知识块组织
的 markdown 笔记。**不**逐 slide 镜像。用户会审,准确性优先于 polish。

## When to trigger

- "整理这份 lecture" / "处理这节课" / "把这份 ppt 变成笔记"
- 用户附 `.pptx` / `.pdf` / 截图 / 课程文本

不应触发:tutorial(走 ingest-tutorial)/ 模糊"summarize this"(先问)。

## Inputs

- Lecture 材料(PPT / PDF / 文本 / 图片)
- **课程代码** + **周次** + **lecture 编号**(缺则问一次,不猜)
- 可选:日期 / 标题

## Outputs

1. `01_Projects/<CODE>_中文名/L##_topic_snake.md`(1 个文件)
2. 更新 `01_Projects/<CODE>_中文名/index.md`(MOC,增量追加 Week 段落)

## Dependencies

启动时读:
- `06_Metadata/Templates/lecture-topic.md`(笔记模板)
- `01_Projects/<CODE>_中文名/index.md`(MOC,若存在)

工具:
- `.scripts/mineru_extract.py`(PDF → markdown + images,用 MinerU API)
- `.scripts/extract_images.py`(逐页 PNG 渲染,供 vision 核对;PPT 也用)
- `.env` 含 `MINERU_API_TOKEN`(.gitignored,从 https://mineru.net/apiManage/token 申请)

## Workflow

### Step 1: 加载上下文

1. 检查 `01_Projects/<CODE>_中文名/` 是否存在;若不存在,**问用户**课程
   中文名 + 学期,创建文件夹 + 桩 `index.md`(MOC)。
2. 读 MOC 看历史(domain_tags / 之前的 Week)。
3. 加载 `06_Metadata/Templates/lecture-topic.md`。

### Step 2: PDF/PPT 处理(MinerU 优先 + vision 兜底)

#### 2a: PDF — 用 MinerU API 抽 markdown(基础底稿)

```bash
py .scripts/mineru_extract.py "<pdf_path>" "01_Projects/<CODE>_中文名/_attachments/" --model vlm
```

输出:
- `_attachments/<pdf_stem>/full.md` — MinerU 抽的 markdown(含公式 + 表格)
- `_attachments/<pdf_stem>/images/*.jpg` — MinerU 抽出的图片

如果 MinerU 失败(token 过期 / 配额超 / 超时 / 网络),**fallback 到纯 vision 模式**(跳过 2a,只跑 2b)。

#### 2b: 逐页渲染 PNG(供 vision 核对;PPT 唯一处理方式)

```bash
py .scripts/extract_images.py "<source>" "01_Projects/<CODE>_中文名/_attachments/_pages/" --prefix <CODE>_L## --pages --dpi 150
```

**PDF 也跑 2b**(供 vision 对照 MinerU 结果,补 MinerU 漏抽的内容)。
**PPT 直接走 2b**(MinerU 不支持 .pptx)。
文本 / 截图跳过整个 Step 2。

### Step 3: 提取 + 核对(MinerU 底稿 + vision 补漏 + 图片描述)

#### 3a: 读 MinerU markdown(若 PDF + 2a 成功)

Read `_attachments/<pdf_stem>/full.md`,作为内容基础底稿。

#### 3b: vision 核对 + 图片重命名 + 嵌入推荐(sub-agent 三块输出)

启动多个 sub-agent 并行,每个负责一段页码:

| PDF 页数 | Agent 数 | 每个 Agent 负责 |
|---|---|---|
| ≤15 页 | 2 个 | 6-8 页 |
| 16-40 页 | 3 个 | 8-13 页 |
| >40 页 | 4 个 | 10-15 页 |

每个 Agent 读 `_attachments/_pages/<page>.png` + MinerU md 对应段,**输出三块**:

**块 1 — 逐页核对**(每页一段):

```
### Slide X: [页面标题]
**MinerU 已抽**: (从 full.md 对应段落复制)
**vision 补漏**: (MinerU 漏的文字 / 公式 / callout)
**误识修正**: (公式 / 字符纠正)
**本页图表**: (描述 + 引用 hash)
```

**块 2 — 图片重命名清单**(本段所有 MinerU 抽出的图,**全部都要列**):

| 旧 hash | 新文件名 | 类型 |
|---|---|---|
| `45f8...cd97` | `<CODE>_L##_p<2位页码>_<topic-slug>_<type>.jpg` | concept |

`type` 取值:`concept` / `data` / `flow` / `formula` / `table` / `decor`

**块 3 — 嵌入推荐清单**(只列推荐度 ≥ 3):

| 新文件名 | 推荐度 | 理由 | 建议放哪个知识块 |
|---|---|---|---|
| ... | 5 | ... | ... |

推荐度尺度:5 必嵌 / 4 强烈建议 / 3 可嵌可不嵌 / 2-1 不必列。

**纯 vision 模式**(2a 失败时):无块 2/3(没 MinerU 图),只输出块 1。

#### 3c: 主线程聚合(批量 mv + verify Read)

收齐所有 sub-agent 报告后,**主线程做**:

1. **批量 mv**:按"块 2 重命名清单"把 `_attachments/<pdf_stem>/images/` 下的 hash 文件名改成语义名。
2. **verify Read**:对"块 3"中**推荐度 ≥ 4** 的图,Read 一次亲眼看 — 确认 sub-agent 描述准、值得嵌。每张图打三种 verdict:
   - **VERIFIED 嵌入** — 进笔记
   - **VERIFIED 跳过** — 不进笔记(sub-agent 推荐度高估)
   - **追加嵌入** — sub-agent 没推荐但 Read 时发现值得嵌

#### 3d: 覆盖率确认

每页 slide 都必须在块 1 里出现。缺失就 Read 补读。

### Step 3.5: 备份 MinerU markdown + 清理临时页

```bash
# 1. 备份 MinerU 原始 markdown(防 _attachments/<pdf_stem>/ 误删丢失内容)
mkdir -p 01_Projects/<CODE>_中文名/_attachments/_mineru_md/
cp "01_Projects/<CODE>_中文名/_attachments/<pdf_stem>/full.md" \
   "01_Projects/<CODE>_中文名/_attachments/_mineru_md/L##_full.md"

# 2. 清理逐页 PNG(供 vision 用,vision 跑完不再需要)
rm -rf 01_Projects/<CODE>_中文名/_attachments/_pages/
```

**保留** `_attachments/<pdf_stem>/`(MinerU 抽的 markdown + 已重命名的图)。

### Step 4: 生成笔记

**先看目标 `L##_<topic>.md` 是否已存在且非空**(增量 vs 重写决策):

- **已存在且非空**:
  1. Read 旧版列章节清单
  2. **优先 Edit 增量改动**(加图、修 OCR 错、补漏)— **不 Write 重写**
  3. 仅在用户明确说"重写"或旧版质量不可接受时才 Write
  4. Write 前列「v2 章节 vs v1 章节」对照清单,**v2 章节数 ≥ v1 章节数**
- **不存在或空**:按下面模板从零写。

按 `06_Metadata/Templates/lecture-topic.md` 模板填:

- **frontmatter**:按模板字段写(`lecture: L##`)
- **知识结构图**:Mermaid 图展示本讲核心概念关系
- **知识块**:**按逻辑主题组织,不按 slide 编号**
  - 每个知识块 = 一个独立可理解的知识点
  - 公式用 LaTeX + 一行中文解读
  - **目标读者 = 高中毕业生**,笔记要脱离 PPT 后独立可读
- **术语定义**:首次术语必须就地定义 1-2 句(若 PPT 没解释)
- **图片嵌入**:`![[file.png]]` 紧跟知识块,有信息量加一句 `> 说明`
- **内容层次**:
  - PPT 原文 → 折叠在末尾 `> [!info]-` callout
  - AI 解释 → 正文知识块,无标记
  - AI 延伸 → `> [!tip] 延伸(非 PPT 内容)` callout
- **疑问**:3-5 个具体可操作的问题,放 `## 我的疑问` 段

### Step 5: 自检

- **slide 覆盖率**:每页核心信息是否在某知识块?未覆盖标"跳过(装饰
  /过渡)"或补充。报告 `Slides 覆盖: 18/20`。
- **术语自洽**:首次术语是否已定义?未定义补完。

### Step 6: 增量 MOC 更新

`index.md` **追加** Week 段落(不改已有):

```markdown
## Week {{WEEK}}

{{一两句概要}}

- [[L##_topic_snake]]

**本周疑问**:
- (汇总笔记里的 1-3 个关键疑问)
```

**Append-only**:不修改已有 Week 段落。

### Step 7: 报告

```markdown
## Ingestion complete: <CODE> Week ## L##

**笔记**: [[L##_topic_snake]]
**Slides 覆盖**: X/Y(跳过 Z 页:课程信息页/装饰页)
**术语自洽**: N/N(全部已定义或已链接)
**图片**: N 张嵌入
**疑问汇总**:
- (笔记里的疑问)
```

## Rules

1. **不编造 slide 内容** — PPT 文字稀疏时写"原文未明",不要自己编
2. **解读长度匹配 slide 密度** — 一行 slide 一行解读。术语首次定义除外
3. **AI 延伸必须用 `> [!tip] 延伸(非 PPT 内容)` callout** — 不混进正文
4. **MOC 更新 append-only** — 不改已有 Week 段落

## Example

**Good**(知识块 + 图):
```markdown
## 知识块 1 — Fick 第一定律

扩散通量与浓度梯度成正比:

$$J_A = -D_{AB} \frac{dc_A}{dz}$$

**解读**: A 组分通量 = 扩散系数 × 浓度梯度的负值。负号表示
扩散从高浓度到低浓度方向。

[[扩散系数]] $D_{AB}$ 量级决定传质速率——气体 ~10⁻⁵ m²/s,
液体 ~10⁻⁹ m²/s。

![[CME222_L02_s09.png]]
> Fick 定律示意:通量方向沿浓度降低方向

> [!tip] 延伸(非 PPT 内容)
> Fick 定律与傅里叶导热 $q = -k\nabla T$ 形式完全类比。
```

**Bad**:全英文 / slide-by-slide 镜像 / 没有知识块组织 / 通用 padding 替代具体解读。

## Failure modes

| 模式 | 触发 | 处理 |
|---|---|---|
| PDF 读 password-protected 误报 | Read 工具说 PDF 加密但实际未加密 | 改用 `PyMuPDF`(`import fitz`)提取 |
| 课程文件夹不存在 | `01_Projects/<CODE>_中文名/` 没有 | 问用户中文名 + 学期,创建文件夹 + index.md,**不擅自猜** |
| extract_images.py 失败 | PPT 解析错 / 0 图 | WARN,继续生成笔记(无图嵌入),报告用户 |
| MOC 已存在但 frontmatter 不合规 | index.md 缺 frontmatter | WARN,只追加 Week 段,**不修 frontmatter** |
| 一讲多 PDF/PPT | 用户传多份附件 | 逐份提取后合并到**同一笔记**;不为每份生成独立笔记 |
| sub-agent 报告主导写笔记 | 主线程拿到 sub-agent 输出 + MinerU md 后**没 Read 旧笔记 / 模板**就动笔 | **STOP**,回到 Step 4 开头先 Read 旧笔记列章节清单,再决定 Edit 还是 Write |
