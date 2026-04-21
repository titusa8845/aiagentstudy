# Gemini CLI + OpenCode 共用 Office-Word-MCP-Server（Windows）

更新日期：2026-04-21

## 目標

讓 `gemini cli`、`opencode`（以及 `codex`）共用同一個本機 MCP Server：

- Server 名稱：`word-document-server`
- Transport：`stdio`
- Python Server：`Office-Word-MCP-Server`

---

## 1) 安裝位置（英文路徑）

使用路徑：

`C:\Users\user\mcp\Office-Word-MCP-Server-main`

---

## 2) 安裝（venv 方式）

在 `C:\Users\user\mcp\Office-Word-MCP-Server-main` 執行：

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip install .
```

快速確認可啟動：

```powershell
.\.venv\Scripts\python.exe -m word_document_server.main
```

---

## 3) Codex 設定

檔案：`C:\Users\user\.codex\config.toml`

加入：

```toml
[mcp_servers.word-document-server]
command = "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main\\.venv\\Scripts\\python.exe"
args = ["-m", "word_document_server.main"]

[mcp_servers.word-document-server.env]
PYTHONPATH = "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main"
MCP_TRANSPORT = "stdio"
```

---

## 4) Gemini CLI 設定

檔案：`C:\Users\user\.gemini\settings.json`

在 `mcpServers` 內加入：

```json
"word-document-server": {
  "command": "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main\\.venv\\Scripts\\python.exe",
  "args": [
    "-m",
    "word_document_server.main"
  ],
  "env": {
    "PYTHONPATH": "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main",
    "MCP_TRANSPORT": "stdio"
  }
}
```

---

## 5) OpenCode 設定

檔案：`C:\Users\user\.config\opencode\opencode.json`

在 `mcp` 內加入：

```json
"word-document-server": {
  "type": "local",
  "enabled": true,
  "command": [
    "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main\\.venv\\Scripts\\python.exe",
    "-m",
    "word_document_server.main"
  ],
  "environment": {
    "PYTHONPATH": "C:\\Users\\user\\mcp\\Office-Word-MCP-Server-main",
    "MCP_TRANSPORT": "stdio"
  }
}
```

---

## 6) 共用驗證

### Codex

```powershell
codex.cmd mcp list
```

預期：`word-document-server` 為 `enabled`。

### Gemini CLI

```powershell
gemini.cmd -d mcp list
```

預期：`word-document-server ... Connected`。

### OpenCode

```powershell
opencode.cmd mcp list
```

預期：`word-document-server connected`。

---

## 7) 功能驗證（非只看連線）

### 驗證 A：列出工具

```powershell
.\.venv\Scripts\python.exe .\test_word_mcp_list_tools.py
```

本次結果：`TOOLS_COUNT 54`，包含 `get_document_text`。

### 驗證 B：讀取 docx

```powershell
.\.venv\Scripts\python.exe .\test_word_mcp_read_docx.py
```

本次結果：
- 產生 `word_mcp_test.docx`
- 成功讀出內容（標題與兩段測試文字）

---

## 8) 這台機器的最終狀態（已完成）

- `ppt-server` 與 `word-document-server` 皆已在：
  - `codex`
  - `gemini cli`
  - `opencode`
  連線成功並可用。

