# Workflow B — 每周回顾 (Weekly Review)

> 脱敏版。敏感信息已替换为 `<占位符>`。

## 节点链路

```
Schedule(周六8点) → Code(算日期) → Notion(查询本周) → Code(聚合) → LLM Chain(AI分析) → Code(解析) → Notion(写入回顾)
                                                                        └─ DeepSeek pro (subnode)
```

## 关键代码

### Schedule 触发器
每周触发，`field: weeks`, `triggerAtDay: [6]`（周六）, `triggerAtHour: 8`。

### 算起始日期 (Code, runOnceForAllItems)

```javascript
const nowUtc = new Date();
const cst = new Date(nowUtc.getTime() + 8 * 60 * 60 * 1000);
const pad = (n) => String(n).padStart(2, "0");
const start = new Date(cst.getTime() - 7 * 24 * 60 * 60 * 1000);
const startDate = `${start.getUTCFullYear()}-${pad(start.getUTCMonth()+1)}-${pad(start.getUTCDate())}`;
const end = new Date(cst);
const fmt = (d) => `${d.getUTCMonth()+1}/${d.getUTCDate()}`;
const startDisp = new Date(cst.getTime() - 6 * 24 * 60 * 60 * 1000);
const dateRange = `${fmt(startDisp)}-${fmt(end)}`;
const genDate = `${cst.getUTCFullYear()}-${pad(cst.getUTCMonth()+1)}-${pad(cst.getUTCDate())}`;
return [{ json: { startDate, dateRange, genDate } }];
```

### 查询本周感悟 (Notion getAll)

- Database ID: `<daily-db-id>`，凭证 `<notion-credential>`
- **`simple: false`**（关键：true 会丢失正文字段）
- Filter: `日期` `on_or_after` `{{ $json.startDate }}`（避开 past_week bug）

### 聚合本周记录 (Code, runOnceForAllItems)

健壮字段提取，兼容顶层与 properties 嵌套，并过滤空记录：

```javascript
const items = $input.all();
const meta = $('算起始日期').first().json;

function extractText(prop) {
  if (prop == null) return "";
  if (typeof prop === "string") return prop;
  if (typeof prop === "number") return String(prop);
  if (Array.isArray(prop)) {
    return prop.map(p => {
      if (typeof p === "string") return p;
      if (p && p.plain_text) return p.plain_text;
      if (p && p.text && p.text.content) return p.text.content;
      if (p && p.name) return p.name;
      return "";
    }).filter(Boolean).join("、");
  }
  if (typeof prop === "object") {
    if (prop.plain_text) return prop.plain_text;
    if (prop.text && prop.text.content) return prop.text.content;
    if (prop.name) return prop.name;
    if (Array.isArray(prop.rich_text)) return extractText(prop.rich_text);
    if (Array.isArray(prop.title)) return extractText(prop.title);
    if (Array.isArray(prop.multi_select)) return extractText(prop.multi_select);
  }
  return "";
}

function getField(j, name) {
  if (j == null) return "";
  if (j[name] !== undefined) return extractText(j[name]);
  if (j.properties && j.properties[name] !== undefined) return extractText(j.properties[name]);
  return "";
}

const valid = [];
for (const it of items) {
  const j = it.json;
  const title = getField(j, "标题") || getField(j, "name");
  const content = getField(j, "内容");
  const summary = getField(j, "摘要");
  const tags = getField(j, "标签");
  if ((content && content.trim()) || (summary && summary.trim())) {
    valid.push({ title, content: content || summary, tags });
  }
}

const count = valid.length;
if (count === 0) {
  return [{ json: { isEmpty: true, count: 0, dateRange: meta.dateRange, genDate: meta.genDate } }];
}
const lines = valid.map((v, i) => `${i+1}. 【${v.title}】${v.content}${v.tags ? "（标签：" + v.tags + "）" : ""}`).join("\n");
return [{ json: { isEmpty: false, count, records: lines, dateRange: meta.dateRange, genDate: meta.genDate } }];
```

### AI 周回顾 (LLM Chain + DeepSeek pro)

模型：`deepseek-v4-pro`，`responseFormat: json_object`，`temperature: 0.4`

Prompt 要求：基于真实记录、结构化分点（一句话概述 + 1/2/3 分点 + 值得关注）、专业克制、禁止情绪化抒情。输出 `review/themes/title`。

### 写入回顾 (Notion)

- Database ID: `<weekly-db-id>`，凭证 `<notion-credential>`
- 字段：标题、日期范围、回顾正文、本周主题(multi_select)、生成日期、记录条数(number)
