---
name: inbox-processor
description: Help organize and process items in the 00_Inbox folder according to the PARA method. Use when the user asks to process, clear, sort, triage, or organize their inbox. / 按 PARA 方法整理和处理 00_Inbox 里的条目。当用户说"处理 inbox"、"清空 inbox"、"分类整理"、"triage 一下" 时触发。
---

# Inbox Processor

按 PARA 方法整理和处理 00_Inbox 里的条目。

## 任务

看下 `00_Inbox/` 里所有笔记，帮做分类：

1. **扫一遍 Inbox**
   - 列出 00_Inbox 里所有文件
   - 排除 README.md 和 Welcome.md

2. **逐条分析**
   - 读内容
   - 识别笔记类型
   - 给出合适的去处建议

3. **分类规则**
   - **→ 01_Projects**：有截止日期，有具体产出
   - **→ 02_Areas**：持续性责任，没结束日期
   - **→ 03_Resources**：参考资料、知识
   - **→ 04_Archive**：旧的 / 完成的，已不活跃
   - **→ 删除**：没价值、重复、临时的

4. **建议动作**

   ```
   文件: [filename]
   类型: [识别出的类型]
   去处: [建议的目录]
   理由: [为啥这么分]
   关联: [它能链上的已有笔记]
   ```

5. **识别模式**
   - 多笔记之间反复出现的主题
   - 可以合并的笔记
   - 条目之间缺失的连接

## 输出格式

给一份清楚的行动计划：

1. 要移动的条目（含去处）
2. 要合并或链接的条目
3. 要删除的条目
4. 需要更多上下文的条目

## 记住

- 有些条目正当地属于 Inbox（每日笔记、临时记录）
- 别过度组织 — 有时候"够用"就完美
- 找连接想法的机会，不只是归档
