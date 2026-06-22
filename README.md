# Voice Reflection · 语音感悟记录工具

> 一个用语音随手记录思考、由 AI 自动润色归档、并按周生成回顾的个人工具。
> A personal tool to capture thoughts by voice, auto-polish & archive them with AI, and generate weekly reviews.

由 **老柒 (Seven)** 构建 · Part of [seven-ai-lab](https://github.com/evilsinme7/seven-ai-lab)

**当前版本 / Current: v1.1** · 迭代历史 / History → [CHANGELOG](./CHANGELOG.md)

---

## 中文

### 这是什么

一个"随想随记"的个人感悟工具。在 iPhone 上一键唤起语音听写，说一句话，可顺手编辑微调，内容会被自动润色（加标点、改错别字、修正中英混杂识别、提炼标题与标签），归档到 Notion 数据库；每周六自动生成一份结构化的"本周思考回顾"。

核心理念：**记录的摩擦力要尽可能低（一键说话），但沉淀的质量要尽可能高（AI 润色 + 周回顾）。**

### 架构

```
┌─────────────┐     ┌──────────────────────── n8n Workflow A ────────────────────────┐
│  iOS 快捷指令 │     │                                                                  │
│  语音听写     │     │  Webhook(鉴权) → 整理字段 → AI润色+纠错(DeepSeek) → 解析 → 写入   │
│  → 编辑微调   │────▶│                                                                  │
│  (背面轻点)   │ POST│                                                                  │
└─────────────┘     └──────────────────────────────────────────────────────────────┘
                                                                            │
                                                                            ▼
                                                              ┌──────────────────────┐
                                                              │   Notion 每日感悟库    │
                                                              └──────────────────────┘
                                                                            │
┌──────────────────────── n8n Workflow B (每周六定时) ────────────────────────┐
│  定时触发 → 算日期 → 查询本周记录 → 聚合 → AI分析(DeepSeek) → 写入每周回顾库   │
└──────────────────────────────────────────────────────────────────────────┘
```

### 技术栈

- **入口**：iOS 快捷指令（听写 + 编辑微调 + 获取URL内容）
- **编排**：n8n（自托管）
- **AI**：DeepSeek V4（随手记润色用 flash，周回顾分析用 pro）
- **存储**：Notion（每日感悟库 + 每周回顾库两个数据库）
- **鉴权**：Webhook Header Auth

### 设计决策

记录几个关键的取舍，这些比代码本身更值得留存：

1. **为什么不用现成的开源方案？**
   调研过 voice-to-notion、WhisperJournal 等开源项目，但它们都引入额外部署负担（常驻服务监控文件夹、或自编译 App）。而本方案复用 iOS 自带听写 + 已有的 n8n，"语音转文字"这个环节成本几乎为零。结论是"借模式、不借代码"。

2. **为什么把 JSON 拼接放到服务端？**
   最初让快捷指令拼 JSON，但这是手机端最易出错的一步。改为让快捷指令直接发纯文本、由 n8n 端做解析，手机端只剩"说话 + 发送"两个动作，大幅降低配置门槛。

3. **随手记用 flash，周回顾用 pro。**
   随手记是轻量润色任务，flash 足够且快（~5秒）；周回顾需要综合一整周内容做深度提炼，值得用 pro。按需选型，不盲目堆算力。

4. **原话与润色稿都保留。**
   口语原文有其价值（最真实的表达），润色稿便于回看。两者分字段存储。

### 踩坑记录

真实开发中遇到的坑，留作备忘：

- **草稿 vs 发布版**：n8n 通过 API/MCP 修改 workflow 只更新草稿，必须手动 Publish 才会生效到生产环境。调试时若改动"不生效"，先确认是否发布。
- **快捷指令的 body 结构**：用"文件"模式发送时，n8n 收到的不是纯字符串，而是 `body.data`。取值需兼容多种结构。
- **Notion getAll 的 simple 模式**：`simple:true` 会把记录裁剪到只剩 id/name/url，正文字段全丢。需用 `simple:false` 取完整数据，再做健壮的字段提取。
- **Notion 日期筛选的 past_week bug**：n8n Notion 节点处理 `past_week` 相对条件时会错误塞入时间戳导致报错。改用 `on_or_after` + 明确算出的日期更稳。
- **时区**：不要依赖 n8n 实例的系统时区，在代码里强制按目标时区（UTC+8）计算，否则时段判断和日期会偏。

### 复现指南

见 [`SETUP.md`](./SETUP.md)。注意所有配置中的 URL、数据库 ID、凭证名均为占位符，需替换为你自己的。

---

## English

### What is this

A personal reflection tool for capturing thoughts on the fly. Trigger voice dictation on your iPhone with one tap, speak a thought, optionally tweak it on the spot, and it gets auto-polished (punctuation, typo fixes, mixed Chinese-English correction, title & tag extraction) and archived to a Notion database. Every Saturday, it auto-generates a structured weekly thinking review.

Core idea: **minimize the friction of capture (one-tap voice), maximize the quality of what's retained (AI polish + weekly review).**

### Tech Stack

- **Entry**: iOS Shortcuts (Dictate + on-the-spot edit + Get Contents of URL)
- **Orchestration**: n8n (self-hosted)
- **AI**: DeepSeek V4 (flash for note polishing, pro for weekly analysis)
- **Storage**: Notion (two databases: daily reflections + weekly reviews)
- **Auth**: Webhook Header Auth

### Design Decisions

1. **Why not an existing open-source solution?** Surveyed projects like voice-to-notion and WhisperJournal, but they add deployment overhead. Reusing iOS built-in dictation + existing n8n makes the speech-to-text step essentially free. "Borrow the pattern, not the code."
2. **Why move JSON assembly server-side?** The phone shouldn't assemble JSON — it's the most error-prone step. The shortcut sends plain text; n8n parses it. The phone is left with just "speak + send."
3. **flash for notes, pro for reviews.** Match the model to the task weight.
4. **Keep both raw and polished text.** The spoken original has value; the polished version aids review.

### Lessons Learned

- **Draft vs. published**: n8n workflow edits via API/MCP only update the draft — you must Publish for changes to take effect in production.
- **Shortcut body structure**: Sending via "file" mode, n8n receives `body.data`, not a plain string.
- **Notion getAll simple mode**: `simple:true` strips records to id/name/url only. Use `simple:false` for full data.
- **Notion past_week bug**: The n8n Notion node mishandles `past_week`. Use `on_or_after` with an explicit date instead.
- **Timezone**: Don't rely on the n8n instance's system timezone; compute against your target zone (UTC+8) explicitly.

### Setup

See [`SETUP.md`](./SETUP.md). All URLs, database IDs, and credential names in the configs are placeholders.

---

*License: MIT*
