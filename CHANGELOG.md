# 更新日志 / Changelog

本项目所有重要变更都记录在此。
All notable changes to this project are documented here.

格式参考 [Keep a Changelog](https://keepachangelog.com/)，版本遵循语义化版本。

---

## [v1.2] - 2026-06-23

### 中文

**主题：降低复刻门槛，让他人可快速搭建**

#### 新增 (Added)
- **可导入的 workflow JSON**：在 `workflows/` 下提供两条 workflow 的脱敏 JSON 文件（`workflow-a-capture.json`、`workflow-b-weekly-review.json`）。他人可在自己的 n8n 中通过 "Import from File" 一键导入节点结构，无需照着文档手动重搭。
- **快速复刻指南**：README 新增 "快速复刻 / Quick Start" 段落，列出从导入到运行的最短路径。

#### 说明
- 所有 JSON 中的数据库 ID、webhook ID、凭证引用均替换为占位符（`<...>` / `REPLACE_WITH_...`），导入后需替换为自己的配置。

### English

**Theme: Lower the barrier to replication**

#### Added
- **Importable workflow JSON**: Sanitized JSON for both workflows under `workflows/`. Others can import the node structure directly via "Import from File" in their own n8n.
- **Quick Start guide**: A new section in README outlining the shortest path from import to running.

---

## [v1.1] - 2026-06-22

### 中文

**主题：提升语音转写质量与可控性**

针对实际使用中发现的"iOS 自带转写对中英混杂识别不准"问题，本次迭代加入两道防线：

#### 新增 (Added)
- **转写后编辑步骤**：快捷指令在听写与发送之间，新增一个预填转写结果的可编辑文本框。说完轻点结束时，可顺手微调专有名词、修正英文识别错误，不想改则直接发送 —— 不增加额外操作负担。

#### 优化 (Changed)
- **AI 润色强化纠错**：润色 prompt 明确告知 DeepSeek 输入来自语音转写，可能存在同音字错误、中英混杂时英文单词或专有名词被错误识别等问题，要求其结合上下文修正。形成"人工编辑 + AI 纠错"的双重保障。

#### 设计说明
- 这次没有引入更重的方案（如换专业语音识别 API、录音上传转写），而是用最小改动解决问题。判断依据：用户原本就需要"轻点结束听写"，将这一刻顺势变为"可编辑"，几乎零额外成本，却同时满足了"编辑能力"和"纠错"两个诉求。

### English

**Theme: Improving transcription quality and controllability**

To address the real-world issue of iOS built-in dictation mishandling mixed Chinese-English input, this iteration adds two safeguards:

#### Added
- **Post-transcription edit step**: The shortcut now shows an editable text box (pre-filled with the transcription) between dictation and sending. Tweak proper nouns or fix misrecognized English on the spot, or just send as-is — no extra friction.

#### Changed
- **Stronger AI correction**: The polishing prompt now tells DeepSeek the input is from speech recognition and may contain homophone errors or misrecognized English/proper nouns, asking it to correct using context. This forms a "human edit + AI correction" double safeguard.

---

## [v1.0] - 2026-06-16

### 中文

**首个版本：完整的语音感悟记录系统**

#### 核心功能
- **随手记 (Workflow A)**：iOS 快捷指令一键语音听写 → n8n webhook（Header Auth 鉴权）→ DeepSeek V4 Flash 润色（加标点、改错别字、生成标题/标签/摘要）→ 写入 Notion 每日感悟库。
- **每周回顾 (Workflow B)**：每周六定时触发 → 查询过去 7 天记录 → DeepSeek V4 Pro 生成结构化分点的理性周回顾 → 写入 Notion 每周回顾库。
- **一键唤起**：支持主屏图标、背面轻点、Action Button。
- **本地时区**：强制按 UTC+8 计算时段与日期。

### English

**Initial release: A complete voice reflection system**

#### Core Features
- **Capture (Workflow A)**: One-tap iOS dictation → n8n webhook (Header Auth) → DeepSeek V4 Flash polishing → Notion daily database.
- **Weekly Review (Workflow B)**: Saturday schedule → query past 7 days → DeepSeek V4 Pro structured review → Notion weekly database.
- **Quick triggers**: Home screen icon, Back Tap, Action Button.
- **Local timezone**: Forced UTC+8 computation.
