# iOS 快捷指令配置 / iOS Shortcut Setup

> v1.1：在听写与发送之间新增可编辑步骤。
> v1.1: Added an editable step between dictation and sending.

## 三个动作 / Three actions

### 1. 听写文本 / Dictate Text
- 语言 / Language: 中文（中国）
- 停止聆听 / Stop Listening: **在轻点时 / On Tap**（说完自己点屏幕结束，不会被自然停顿打断）

### 2. 请求文本（编辑步骤）/ Ask for Text (edit step)
- 默认回答 / Default Answer: 插入"听写的文本"变量 / Insert the "Dictated Text" variable
- 允许多行 / Allow Multiple Lines: 开启 / On
- 效果：弹出预填转写结果的编辑框，可微调专有名词、修正英文识别错误；不改则直接发送。
- Effect: Shows an editable box pre-filled with the transcription. Tweak proper nouns or fix English errors, or send as-is.

### 3. 获取URL内容 / Get Contents of URL
- URL: `<your-webhook-url>`
- 方法 / Method: **POST**
- 请求体 / Request Body: **文件 / File**
- 文件 / File: 插入"**请求输入**"变量（即上一步编辑后的文本，而非原始听写）
  / Insert the "**Provided Input**" variable (the edited text from step 2, NOT the raw dictation)
- 头部 / Headers: `<your-auth-header-name>` = `<your-secret>`

> ⚠️ 关键：第 3 步的"文件"必须用"请求输入"（编辑后的文本），否则你的编辑不会生效。
> ⚠️ Critical: Step 3's "File" must use "Provided Input" (edited text), or your edits won't take effect.

## 一键唤起 / Quick Trigger
- 添加到主屏幕 / Add to Home Screen
- 或：设置 → 辅助功能 → 触控 → 轻点背面 → 选此快捷指令
  / Settings → Accessibility → Touch → Back Tap → select this shortcut
