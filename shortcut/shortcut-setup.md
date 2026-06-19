# iOS 快捷指令配置 / iOS Shortcut Setup

## 两个动作即可 / Just two actions

### 1. 听写文本 / Dictate Text
- 语言 / Language: 中文（中国）
- 停止聆听 / Stop Listening: **在轻点时 / On Tap**（说完自己点屏幕结束，不会被自然停顿打断）

### 2. 获取URL内容 / Get Contents of URL
- URL: `<your-webhook-url>`
- 方法 / Method: **POST**
- 请求体 / Request Body: **文件 / File**
- 文件 / File: 插入"听写的文本"变量 / Insert the "Dictated Text" variable
- 头部 / Headers: `<your-auth-header-name>` = `<your-secret>`

## 一键唤起 / Quick Trigger
- 添加到主屏幕 / Add to Home Screen
- 或：设置 → 辅助功能 → 触控 → 轻点背面 → 选此快捷指令
  / Settings → Accessibility → Touch → Back Tap → select this shortcut
