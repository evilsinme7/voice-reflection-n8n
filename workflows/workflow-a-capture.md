# Workflow A — 随手记 (Capture)

> 脱敏版。敏感信息已替换为 `<占位符>`。基于 n8n Workflow SDK 描述。

## 节点链路

```
Webhook(接收感悟) → Code(整理字段) → LLM Chain(AI润色) → Code(解析结果) → Notion(写入)
                                          └─ DeepSeek flash (subnode)
```

## 关键代码

### 整理字段 (Code, runOnceForEachItem)

```javascript
const b = $json.body;
let raw = "";
if (typeof b === "string") {
  raw = b;
} else if (b && typeof b === "object") {
  raw = b.data || b.text || "";   // 快捷指令"文件"模式发来的是 body.data
}
raw = (raw || "").toString().trim();

// 强制东八区 (UTC+8)，不依赖 n8n 实例时区
const nowUtc = new Date();
const cst = new Date(nowUtc.getTime() + 8 * 60 * 60 * 1000);
const hour = cst.getUTCHours();
const period = hour < 11 ? "早" : (hour < 17 ? "午" : "晚");
const pad = (n) => String(n).padStart(2, "0");
const iso = `${cst.getUTCFullYear()}-${pad(cst.getUTCMonth()+1)}-${pad(cst.getUTCDate())}T${pad(cst.getUTCHours())}:${pad(cst.getUTCMinutes())}:${pad(cst.getUTCSeconds())}+08:00`;

return { raw, period, date: iso };
```

### AI 润色 (LLM Chain + DeepSeek flash)

模型：`deepseek-v4-flash`，`responseFormat: json_object`，`temperature: 0.3`

Prompt（define 模式）：
```
请处理以下语音转写的口语文本，它可能缺少标点、有错别字、表达口语化。

原始文本：{{ $json.raw }}

请输出一个 JSON 对象，包含以下字段：
- "polished"：润色后的文本。加上正确标点、修正错别字、理顺语句，但必须保留原意和说话风格，不要扩写或改变观点。
- "title"：一句话标题，精炼概括核心内容，不超过20字。
- "tags"：标签数组，从 ["AI","商业","投资","健康","生活","灵感"] 中选择1-3个最相关的；如都不合适可新增简短标签。
- "summary"：一两句话的要点摘要。

只返回 JSON，不要任何额外说明或 markdown 代码块标记。
```

### 解析结果 (Code, runOnceForEachItem)

```javascript
const ai = $json;
const prep = $('整理字段').item.json;
let polished = ai.polished;
let title = ai.title;
let tags = ai.tags;
let summary = ai.summary;
// 兜底：万一 AI 输出被包在 text 字符串里
if (polished === undefined && typeof ai.text === "string") {
  try {
    let s = ai.text.trim().replace(/^```json/i, "").replace(/^```/, "").replace(/```$/, "").trim();
    const p = JSON.parse(s);
    polished = p.polished; title = p.title; tags = p.tags; summary = p.summary;
  } catch (e) {}
}
if (!Array.isArray(tags)) tags = [];
polished = polished || prep.raw;
return {
  raw: prep.raw,
  polished,
  title: (title || polished).toString().substring(0, 40) || "（空）",
  summary: summary || "",
  tags: tags.join(","),
  period: prep.period,
  date: prep.date
};
```

### 写入 Notion

- Database ID: `<daily-db-id>`
- 凭证: `<notion-credential>`
- 字段映射：标题→title，原话→raw，内容→polished，摘要→summary，时段→period，日期→date
- 标签：`{{ $json.tags ? $json.tags.split(",").filter(t => t) : [] }}`

## Webhook 鉴权

- Authentication: Header Auth
- 凭证: `<auth-credential>`（Name = `<your-auth-header-name>`, Value = `<your-secret>`）
