# Gemini CLI + OpenCode 共用 Office-PowerPoint-MCP-Server（Windows）

更新日期：2026-04-21

## 目標

讓 `gemini cli`、`opencode`（以及 `codex`）共用同一個本機 MCP Server：

- Server 名稱：`ppt-server`
- Transport：`stdio`
- Python Server：`Office-PowerPoint-MCP-Server`

---

## 1) 建議使用英文路徑（避免中文亂碼）

實際使用路徑：

`C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main`

> 說明：若放在中文路徑，某些 CLI/PowerShell 可能把路徑寫成亂碼，導致 MCP 啟動失敗。

---

## 2) 安裝 Python 依賴（若尚未安裝）

在 repo 根目錄執行：

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

快速檢查 server 可啟動：

```powershell
C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\.venv\Scripts\python.exe C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\ppt_mcp_server.py -h
```

---

## 3) Gemini CLI 設定

### 作法 A：用指令加入（推薦）

```powershell
gemini.cmd mcp add --scope user -t stdio -e PYTHONPATH="C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main" -e MCP_TRANSPORT=stdio ppt-server "C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\.venv\Scripts\python.exe" "C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\ppt_mcp_server.py"
```

### 作法 B：直接確認設定檔

檔案：`C:\Users\user\.gemini\settings.json`

關鍵內容：

```json
{
  "mcpServers": {
    "ppt-server": {
      "command": "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\.venv\\Scripts\\python.exe",
      "args": [
        "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\ppt_mcp_server.py"
      ],
      "env": {
        "PYTHONPATH": "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main",
        "MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

---

## 4) OpenCode 設定

檔案：`C:\Users\user\.config\opencode\opencode.json`

內容：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "ppt-server": {
      "type": "local",
      "enabled": true,
      "command": [
        "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\.venv\\Scripts\\python.exe",
        "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\ppt_mcp_server.py"
      ],
      "environment": {
        "PYTHONPATH": "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main",
        "MCP_TRANSPORT": "stdio"
      }
    }
  }
}
```

---

## 5) 驗證

### Codex

```powershell
codex.cmd mcp list
```

應看到 `ppt-server`，且 `Status = enabled`。

### Gemini CLI

```powershell
gemini.cmd -d mcp list
```

應看到：

- `Configured MCP servers`
- `ppt-server ... (stdio) - Connected`

### OpenCode

```powershell
opencode.cmd mcp list
```

應看到：

- `✓ ppt-server connected`

---

## 6) 目前這台機器的結果（已驗證）

- `codex.cmd mcp list`：`ppt-server` 已 `enabled`
- `gemini.cmd -d mcp list`：`ppt-server` 顯示 `Connected`
- `opencode.cmd mcp list`：`ppt-server` 顯示 `connected`

---

## 7) 常見問題

1. `codex`/`gemini` 在 PowerShell 出現 `.ps1` 被 ExecutionPolicy 擋住  
   請改用 `codex.cmd` / `gemini.cmd` / `opencode.cmd`。

2. 路徑含中文導致亂碼  
   請改用英文路徑（例如 `C:\Users\user\mcp\...`）並重寫設定檔。

3. `WinError 2`  
   檢查：
   - `python.exe` 路徑是否存在
   - `ppt_mcp_server.py` 路徑是否存在
   - `PYTHONPATH` 是否指到 repo 根目錄
