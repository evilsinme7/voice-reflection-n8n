# 复现指南 / Setup Guide

> 所有 URL、数据库 ID、凭证名均为占位符（`<...>`），请替换为你自己的。
> All URLs, database IDs, and credential names are placeholders (`<...>`). Replace with your own.

## 1. Notion 准备

### 每日感悟库 (Daily Reflections)
新建一个 Notion 数据库，字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| 标题 | Title | AI 提炼的一句话标题 |
| 日期 | Date (含时间) | 记录时刻 |
| 标签 | Multi-select | AI 打的主题标签 |
| 内容 | Rich text | AI 润色稿 |
| 原话 | Rich text | 听写原文 |
| 摘要 | Rich text | AI 摘要 |
| 时段 | Select | 早/午/晚/其他 |

### 每周回顾库 (Weekly Reviews)
| 字段 | 类型 |
|------|------|
| 标题 | Title |
| 日期范围 | Rich text |
| 生成日期 | Date |
| 本周主题 | Multi-select |
| 回顾正文 | Rich text |
| 记录条数 | Number |

### Notion Integration
1. 在 notion.so/my-integrations（现为 Connections 页）创建一个 internal integration，类型选 **access token**（非 OAuth）。
2. 勾选 Read / Update / Insert content 权限。
3. **关键**：把这个 integration 加到两个数据库的 Connections 里，否则 n8n 无权访问。

## 2. n8n Workflow A — 随手记

节点链路：`Webhook → Code(整理字段) → LLM Chain(DeepSeek润色) → Code(解析) → Notion(写入)`

要点：
- Webhook：POST，path `<your-path>`，鉴权用 Header Auth。
- 整理字段（Code）：从 `body.data` 取听写文本，强制按 UTC+8 算时段与日期。
- AI 润色：DeepSeek flash，`responseFormat: json_object`，prompt 要求输出 `polished/title/tags/summary`。
- 解析：直接读 AI 展开的字段，带 JSON 字符串兜底。
- 写入 Notion：映射到对应字段，标签用逗号分隔字符串再 split 成数组。

完整脱敏代码见 [`workflows/workflow-a-capture.md`](./workflows/workflow-a-capture.md)。

## 3. n8n Workflow B — 每周回顾

节点链路：`Schedule(周六8点) → Code(算日期) → Notion(查询本周) → Code(聚合) → LLM Chain(DeepSeek分析) → Code(解析) → Notion(写入回顾)`

要点：
- 查询用 `simple: false` + `on_or_after` 明确日期（避开 past_week bug）。
- 聚合节点用健壮的字段提取函数，兼容顶层/properties 嵌套结构，并过滤空记录。
- AI 分析：DeepSeek pro，prompt 要求结构化分点输出。

完整脱敏代码见 [`workflows/workflow-b-weekly-review.md`](./workflows/workflow-b-weekly-review.md)。

## 4. iOS 快捷指令

两个动作：
1. **听写文本**：语言设为中文，停止聆听设为"在轻点时"。
2. **获取URL内容**：
   - URL：`<your-webhook-url>`
   - 方法：POST
   - 请求体：文件
   - 文件：插入"听写的文本"变量
   - 头部：加 `<your-auth-header-name>` = `<your-secret>`（与 n8n Header Auth 凭证一致）

一键唤起：添加到主屏幕 / 设置 → 辅助功能 → 触控 → 轻点背面 → 选此快捷指令。

## 占位符清单

| 占位符 | 含义 |
|--------|------|
| `<your-webhook-url>` | 你的 n8n webhook 生产环境地址 |
| `<your-path>` | webhook 路径 |
| `<daily-db-id>` | 每日感悟库 database ID |
| `<weekly-db-id>` | 每周回顾库 database ID |
| `<notion-credential>` | n8n 中 Notion 凭证名 |
| `<deepseek-credential>` | n8n 中 DeepSeek 凭证名 |
| `<auth-credential>` | n8n 中 Header Auth 凭证名 |
| `<your-auth-header-name>` | 鉴权 header 名称 |
| `<your-secret>` | 鉴权密钥 |
