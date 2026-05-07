---
name: obsidian-bases
description: Create and edit Obsidian Bases (.base files) with views, filters, formulas, and summaries. Use when working with .base files, creating database-like views of notes, or when the user mentions Bases, table views, card views, filters, or formulas in Obsidian. / 创建和编辑 Obsidian Bases（.base 文件），含 view、filter、formula、summary。当处理 .base 文件、创建笔记的数据库样视图，或用户提到 Obsidian 的 Bases / table view / card view / filter / formula 时触发。
---

# Obsidian Bases Skill

让兼容 skill 的 agent 能创建和编辑有效的 Obsidian Bases（`.base` 文件），含 view、filter、formula 和所有相关配置。

## 概览

Obsidian Bases 是基于 YAML 的文件，用来在 Obsidian vault 里定义笔记的动态视图。一个 Base 文件可以含多个 view、全局 filter、formula、属性配置、自定义 summary。

## 文件格式

Base 文件用 `.base` 扩展名，含合法 YAML。也可以嵌入到 Markdown 代码块里。

## 完整 Schema

```yaml
# 全局 filter 应用到 base 里所有 view
filters:
  # 可以是单个 filter 字符串
  # 或递归 filter 对象（and / or / not）
  and: []
  or: []
  not: []

# 定义 formula 属性，所有 view 都能用
formulas:
  formula_name: 'expression'

# 配置属性的显示名和设置
properties:
  property_name:
    displayName: "Display Name"
  formula.formula_name:
    displayName: "Formula Display Name"
  file.ext:
    displayName: "Extension"

# 定义自定义 summary 公式
summaries:
  custom_summary_name: 'values.mean().round(3)'

# 定义一个或多个 view
views:
  - type: table | cards | list | map
    name: "View Name"
    limit: 10                    # 可选：限制结果数
    groupBy:                     # 可选：分组
      property: property_name
      direction: ASC | DESC
    filters:                     # 该 view 专用 filter
      and: []
    order:                       # 显示顺序的属性
      - file.name
      - property_name
      - formula.formula_name
    summaries:                   # 把属性映射到 summary 公式
      property_name: Average
```

## Filter 语法

filter 用来缩小结果。可全局应用，也可针对单个 view。

### Filter 结构

```yaml
# 单个 filter
filters: 'status == "done"'

# AND - 所有条件都必须成立
filters:
  and:
    - 'status == "done"'
    - 'priority > 3'

# OR - 任一条件成立即可
filters:
  or:
    - 'file.hasTag("book")'
    - 'file.hasTag("article")'

# NOT - 排除匹配项
filters:
  not:
    - 'file.hasTag("archived")'

# 嵌套 filter
filters:
  or:
    - file.hasTag("tag")
    - and:
        - file.hasTag("book")
        - file.hasLink("Textbook")
    - not:
        - file.hasTag("book")
        - file.inFolder("Required Reading")
```

### Filter 操作符

| 操作符 | 描述 |
|----------|-------------|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于等于 |
| `<=` | 小于等于 |
| `&&` | 逻辑与 |
| `\|\|` | 逻辑或 |
| <code>!</code> | 逻辑非 |

## 属性

### 三种属性类型

1. **笔记属性** — 来自 frontmatter：`note.author` 或 `author`
2. **文件属性** — 文件元数据：`file.name`、`file.mtime` 等
3. **Formula 属性** — 计算值：`formula.my_formula`

### 文件属性参考

| 属性 | 类型 | 描述 |
|----------|------|-------------|
| `file.name` | String | 文件名 |
| `file.basename` | String | 不含扩展名的文件名 |
| `file.path` | String | 完整文件路径 |
| `file.folder` | String | 父目录路径 |
| `file.ext` | String | 文件扩展名 |
| `file.size` | Number | 文件大小（字节） |
| `file.ctime` | Date | 创建时间 |
| `file.mtime` | Date | 修改时间 |
| `file.tags` | List | 文件里所有标签 |
| `file.links` | List | 文件里的内部链接 |
| `file.backlinks` | List | 链向该文件的所有文件 |
| `file.embeds` | List | 笔记里的 embed |
| `file.properties` | Object | 所有 frontmatter 属性 |

### `this` 关键字

- 主内容区：指 base 文件本身
- 嵌入时：指嵌入它的文件
- 侧边栏：指主内容区当前活跃文件

## Formula 语法

formula 从属性计算值。在 `formulas` 段定义。

```yaml
formulas:
  # 简单算术
  total: "price * quantity"
  
  # 条件逻辑
  status_icon: 'if(done, "✅", "⏳")'
  
  # 字符串格式化
  formatted_price: 'if(price, price.toFixed(2) + " dollars")'
  
  # 日期格式化
  created: 'file.ctime.format("YYYY-MM-DD")'
  
  # 复杂表达式
  days_old: '((now() - file.ctime) / 86400000).round(0)'
```

## 函数参考

### 全局函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `date()` | `date(string): date` | 字符串解析成日期。格式：`YYYY-MM-DD HH:mm:ss` |
| `duration()` | `duration(string): duration` | 解析时长字符串 |
| `now()` | `now(): date` | 当前日期时间 |
| `today()` | `today(): date` | 当前日期（时间 = 00:00:00） |
| `if()` | `if(condition, trueResult, falseResult?)` | 条件 |
| `min()` | `min(n1, n2, ...): number` | 最小数 |
| `max()` | `max(n1, n2, ...): number` | 最大数 |
| `number()` | `number(any): number` | 转数字 |
| `link()` | `link(path, display?): Link` | 创建链接 |
| `list()` | `list(element): List` | 不是 list 就包装成 list |
| `file()` | `file(path): file` | 取 file 对象 |
| `image()` | `image(path): image` | 创建图像用于渲染 |
| `icon()` | `icon(name): icon` | 按名取 Lucide 图标 |
| `html()` | `html(string): html` | 渲染为 HTML |
| `escapeHTML()` | `escapeHTML(string): string` | 转义 HTML 字符 |

### 任意类型函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `isTruthy()` | `any.isTruthy(): boolean` | 强制转布尔 |
| `isType()` | `any.isType(type): boolean` | 检查类型 |
| `toString()` | `any.toString(): string` | 转字符串 |

### Date 函数与字段

**字段**：`date.year`、`date.month`、`date.day`、`date.hour`、`date.minute`、`date.second`、`date.millisecond`

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `date()` | `date.date(): date` | 去掉时间部分 |
| `format()` | `date.format(string): string` | 用 Moment.js 模式格式化 |
| `time()` | `date.time(): string` | 取时间字符串 |
| `relative()` | `date.relative(): string` | 人类可读相对时间 |
| `isEmpty()` | `date.isEmpty(): boolean` | 对 date 永远 false |

### 日期算术

```yaml
# 时长单位：y/year/years、M/month/months、d/day/days、
#           w/week/weeks、h/hour/hours、m/minute/minutes、s/second/seconds

# 加减时长
"date + \"1M\""           # 加 1 月
"date - \"2h\""           # 减 2 小时
"now() + \"1 day\""       # 明天
"today() + \"7d\""        # 一周后

# 日期相减得到毫秒差
"now() - file.ctime"

# 复杂时长算术
"now() + (duration('1d') * 2)"
```

### String 函数

**字段**：`string.length`

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `contains()` | `string.contains(value): boolean` | 检查子串 |
| `containsAll()` | `string.containsAll(...values): boolean` | 全部子串都在 |
| `containsAny()` | `string.containsAny(...values): boolean` | 任一子串在 |
| `startsWith()` | `string.startsWith(query): boolean` | 以 query 开头 |
| `endsWith()` | `string.endsWith(query): boolean` | 以 query 结尾 |
| `isEmpty()` | `string.isEmpty(): boolean` | 空或不存在 |
| `lower()` | `string.lower(): string` | 转小写 |
| `title()` | `string.title(): string` | 转 Title Case |
| `trim()` | `string.trim(): string` | 去空白 |
| `replace()` | `string.replace(pattern, replacement): string` | 替换 pattern |
| `repeat()` | `string.repeat(count): string` | 重复字符串 |
| `reverse()` | `string.reverse(): string` | 反转 |
| `slice()` | `string.slice(start, end?): string` | 子串 |
| `split()` | `string.split(separator, n?): list` | 拆成 list |

### Number 函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `abs()` | `number.abs(): number` | 绝对值 |
| `ceil()` | `number.ceil(): number` | 向上取整 |
| `floor()` | `number.floor(): number` | 向下取整 |
| `round()` | `number.round(digits?): number` | 四舍五入到指定位数 |
| `toFixed()` | `number.toFixed(precision): string` | 定点表示 |
| `isEmpty()` | `number.isEmpty(): boolean` | 不存在 |

### List 函数

**字段**：`list.length`

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `contains()` | `list.contains(value): boolean` | 含某元素 |
| `containsAll()` | `list.containsAll(...values): boolean` | 全部元素在 |
| `containsAny()` | `list.containsAny(...values): boolean` | 任一元素在 |
| `filter()` | `list.filter(expression): list` | 按条件过滤（用 `value`、`index`） |
| `map()` | `list.map(expression): list` | 映射元素（用 `value`、`index`） |
| `reduce()` | `list.reduce(expression, initial): any` | 归约成单值（用 `value`、`index`、`acc`） |
| `flat()` | `list.flat(): list` | 扁平化嵌套 list |
| `join()` | `list.join(separator): string` | 拼成字符串 |
| `reverse()` | `list.reverse(): list` | 反转顺序 |
| `slice()` | `list.slice(start, end?): list` | 子 list |
| `sort()` | `list.sort(): list` | 升序排序 |
| `unique()` | `list.unique(): list` | 去重 |
| `isEmpty()` | `list.isEmpty(): boolean` | 无元素 |

### File 函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `asLink()` | `file.asLink(display?): Link` | 转链接 |
| `hasLink()` | `file.hasLink(otherFile): boolean` | 是否链接到另一文件 |
| `hasTag()` | `file.hasTag(...tags): boolean` | 是否含任一 tag |
| `hasProperty()` | `file.hasProperty(name): boolean` | 是否含某属性 |
| `inFolder()` | `file.inFolder(folder): boolean` | 在文件夹或子文件夹里 |

### Link 函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `asFile()` | `link.asFile(): file` | 取 file 对象 |
| `linksTo()` | `link.linksTo(file): boolean` | 是否链接到某文件 |

### Object 函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `isEmpty()` | `object.isEmpty(): boolean` | 无属性 |
| `keys()` | `object.keys(): list` | 键的 list |
| `values()` | `object.values(): list` | 值的 list |

### 正则函数

| 函数 | 签名 | 描述 |
|----------|-----------|-------------|
| `matches()` | `regexp.matches(string): boolean` | 是否匹配 |

## View 类型

### Table View

```yaml
views:
  - type: table
    name: "My Table"
    order:
      - file.name
      - status
      - due_date
    summaries:
      price: Sum
      count: Average
```

### Cards View

```yaml
views:
  - type: cards
    name: "Gallery"
    order:
      - file.name
      - cover_image
      - description
```

### List View

```yaml
views:
  - type: list
    name: "Simple List"
    order:
      - file.name
      - status
```

### Map View

需要纬度 / 经度属性 + Maps 社区插件。

```yaml
views:
  - type: map
    name: "Locations"
    # 经纬度属性的 map 专用设置
```

## 默认 Summary 公式

| 名称 | 输入类型 | 描述 |
|------|------------|-------------|
| `Average` | Number | 数学平均 |
| `Min` | Number | 最小数 |
| `Max` | Number | 最大数 |
| `Sum` | Number | 全部加和 |
| `Range` | Number | Max - Min |
| `Median` | Number | 中位数 |
| `Stddev` | Number | 标准差 |
| `Earliest` | Date | 最早日期 |
| `Latest` | Date | 最晚日期 |
| `Range` | Date | 最晚 - 最早 |
| `Checked` | Boolean | true 值的数量 |
| `Unchecked` | Boolean | false 值的数量 |
| `Empty` | Any | 空值数量 |
| `Filled` | Any | 非空值数量 |
| `Unique` | Any | 唯一值数量 |

## 完整示例

### 任务跟踪 Base

```yaml
filters:
  and:
    - file.hasTag("task")
    - 'file.ext == "md"'

formulas:
  days_until_due: 'if(due, ((date(due) - today()) / 86400000).round(0), "")'
  is_overdue: 'if(due, date(due) < today() && status != "done", false)'
  priority_label: 'if(priority == 1, "🔴 High", if(priority == 2, "🟡 Medium", "🟢 Low"))'

properties:
  status:
    displayName: Status
  formula.days_until_due:
    displayName: "Days Until Due"
  formula.priority_label:
    displayName: Priority

views:
  - type: table
    name: "Active Tasks"
    filters:
      and:
        - 'status != "done"'
    order:
      - file.name
      - status
      - formula.priority_label
      - due
      - formula.days_until_due
    groupBy:
      property: status
      direction: ASC
    summaries:
      formula.days_until_due: Average

  - type: table
    name: "Completed"
    filters:
      and:
        - 'status == "done"'
    order:
      - file.name
      - completed_date
```

### 阅读列表 Base

```yaml
filters:
  or:
    - file.hasTag("book")
    - file.hasTag("article")

formulas:
  reading_time: 'if(pages, (pages * 2).toString() + " min", "")'
  status_icon: 'if(status == "reading", "📖", if(status == "done", "✅", "📚"))'
  year_read: 'if(finished_date, date(finished_date).year, "")'

properties:
  author:
    displayName: Author
  formula.status_icon:
    displayName: ""
  formula.reading_time:
    displayName: "Est. Time"

views:
  - type: cards
    name: "Library"
    order:
      - cover
      - file.name
      - author
      - formula.status_icon
    filters:
      not:
        - 'status == "dropped"'

  - type: table
    name: "Reading List"
    filters:
      and:
        - 'status == "to-read"'
    order:
      - file.name
      - author
      - pages
      - formula.reading_time
```

### 项目笔记 Base

```yaml
filters:
  and:
    - file.inFolder("Projects")
    - 'file.ext == "md"'

formulas:
  last_updated: 'file.mtime.relative()'
  link_count: 'file.links.length'
  
summaries:
  avgLinks: 'values.filter(value.isType("number")).mean().round(1)'

properties:
  formula.last_updated:
    displayName: "Updated"
  formula.link_count:
    displayName: "Links"

views:
  - type: table
    name: "All Projects"
    order:
      - file.name
      - status
      - formula.last_updated
      - formula.link_count
    summaries:
      formula.link_count: avgLinks
    groupBy:
      property: status
      direction: ASC

  - type: list
    name: "Quick List"
    order:
      - file.name
      - status
```

### Daily Notes 索引

```yaml
filters:
  and:
    - file.inFolder("Daily Notes")
    - '/^\d{4}-\d{2}-\d{2}$/.matches(file.basename)'

formulas:
  word_estimate: '(file.size / 5).round(0)'
  day_of_week: 'date(file.basename).format("dddd")'

properties:
  formula.day_of_week:
    displayName: "Day"
  formula.word_estimate:
    displayName: "~Words"

views:
  - type: table
    name: "Recent Notes"
    limit: 30
    order:
      - file.name
      - formula.day_of_week
      - formula.word_estimate
      - file.mtime
```

## 嵌入 Bases

在 Markdown 文件里嵌入：

```markdown
![[MyBase.base]]

<!-- 指定 view -->
![[MyBase.base#View Name]]
```

## YAML 引号规则

- 含双引号的 formula 用单引号包：`'if(done, "Yes", "No")'`
- 简单字符串用双引号：`"My View Name"`
- 复杂表达式里嵌套引号正确转义

## 常见模式

### 按 Tag 过滤
```yaml
filters:
  and:
    - file.hasTag("project")
```

### 按文件夹过滤
```yaml
filters:
  and:
    - file.inFolder("Notes")
```

### 按日期范围过滤
```yaml
filters:
  and:
    - 'file.mtime > now() - "7d"'
```

### 按属性值过滤
```yaml
filters:
  and:
    - 'status == "active"'
    - 'priority >= 3'
```

### 组合多条件
```yaml
filters:
  or:
    - and:
        - file.hasTag("important")
        - 'status != "done"'
    - and:
        - 'priority == 1'
        - 'due != ""'
```

## 参考

- [Bases Syntax](https://help.obsidian.md/bases/syntax)
- [Functions](https://help.obsidian.md/bases/functions)
- [Views](https://help.obsidian.md/bases/views)
- [Formulas](https://help.obsidian.md/formulas)
