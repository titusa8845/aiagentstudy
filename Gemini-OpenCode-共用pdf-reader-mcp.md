# Gemini CLI + OpenCode 共用 pdf-reader-mcp（Windows）

更新日期：2026-04-21

## 目標

讓 `gemini cli`、`opencode`（以及 `codex`）共用同一個本機 MCP Server：

- Server 名稱：`pdf-reader`
- Transport：`stdio`
- Server Command：`npx.cmd -y @sylphx/pdf-reader-mcp`

---

## 1) 先備環境

請確認 Node.js 可用（Windows PowerShell 建議用 `.cmd`）：

```powershell
node -v
npm.cmd -v
npx.cmd -v
```

---

## 2) Codex 設定

檔案：`C:\Users\user\.codex\config.toml`

加入：

```toml
[mcp_servers.pdf-reader]
command = "npx.cmd"
args = ["-y", "@sylphx/pdf-reader-mcp"]
```

---

## 3) Gemini CLI 設定

檔案：`C:\Users\user\.gemini\settings.json`

在 `mcpServers` 內加入：

```json
"pdf-reader": {
  "command": "npx.cmd",
  "args": [
    "-y",
    "@sylphx/pdf-reader-mcp"
  ]
}
```

---

## 4) OpenCode 設定

檔案：`C:\Users\user\.config\opencode\opencode.json`

在 `mcp` 內加入：

```json
"pdf-reader": {
  "type": "local",
  "enabled": true,
  "command": [
    "npx.cmd",
    "-y",
    "@sylphx/pdf-reader-mcp"
  ]
}
```

---

## 5) 共用驗證

### Codex

```powershell
codex.cmd mcp list
```

### Gemini CLI

```powershell
gemini.cmd -d mcp list
```

### OpenCode

```powershell
opencode.cmd mcp list
```

預期：三邊都能看到 `pdf-reader`，且連線成功（Gemini/OpenCode 顯示 `Connected` / `connected`）。

---

## 6) 功能驗證（read_pdf）

本機測試腳本：

`C:\Users\user\mcp\test_pdf_reader_mcp.py`

執行：

```powershell
C:\Users\user\mcp\Office-Word-MCP-Server-main\.venv\Scripts\python.exe C:\Users\user\mcp\test_pdf_reader_mcp.py
```

本次結果：
- `TOOLS_COUNT 1`
- `TOOL read_pdf`
- 成功讀取 `C:\Users\user\mcp\pdf_reader_test.pdf` 第 1 頁文字與 metadata

---

## 7) 已完成狀態（這台機器）

目前三個 MCP 都已共用成功：

- `ppt-server`
- `word-document-server`
- `pdf-reader`

三邊（Codex / Gemini CLI / OpenCode）皆可連線使用。

