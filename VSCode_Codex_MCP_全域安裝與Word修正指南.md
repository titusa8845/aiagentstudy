# VS Code（Codex 外掛）MCP 全域安裝與修正指南

## 1. 目的
- 在 VS Code 的 Codex/Copilot Chat Agent 中，全域啟用 MCP 伺服器。
- 一次設定後，所有專案共用。
- 內容包含：
  - `ppt-server`
  - `word-document-server`
  - `pdf-reader`
  - `excel-server`
  - `word-document-server` 顯示「已停止」的修正步驟

## 2. 全域設定檔位置
Windows 全域路徑：

```text
C:\Users\user\AppData\Roaming\Code\User\mcp.json
```

## 3. 全域 mcp.json 範例
> 直接使用你目前可運作的配置。

```json
{
  "servers": {
    "ppt-server": {
      "command": "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\.venv\\Scripts\\python.exe",
      "args": [
        "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main\\ppt_mcp_server.py"
      ],
      "env": {
        "PYTHONPATH": "C:\\Users\\user\\mcp\\Office-PowerPoint-MCP-Server-main",
        "MCP_TRANSPORT": "stdio"
      }
    },
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
    },
    "pdf-reader": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@sylphx/pdf-reader-mcp"
      ]
    },
    "excel-server": {
      "command": "C:\\Users\\user\\mcp\\excel-mcp-server\\.venv\\Scripts\\excel-mcp-server.exe",
      "args": [
        "stdio"
      ]
    }
  }
}
```

## 4. 生效方式
1. 儲存 `mcp.json`。
2. 完全關閉 VS Code，再重開。
3. 或執行 `Developer: Reload Window`。
4. 到 `MCP 伺服器`清單確認 4 個 server 出現。

## 5. 驗證指令（PowerShell）
```powershell
# 檢查全域設定檔
Get-Content -LiteralPath "C:\Users\user\AppData\Roaming\Code\User\mcp.json"

# 檢查 Node / npx（pdf-reader 需要）
Get-Command node
Get-Command npx.cmd
node -v

# 檢查 Excel MCP
& "C:\Users\user\mcp\excel-mcp-server\.venv\Scripts\excel-mcp-server.exe" --help

# 檢查 Word 模組可載入
& "C:\Users\user\mcp\Office-Word-MCP-Server-main\.venv\Scripts\python.exe" -c "import importlib; importlib.import_module('word_document_server.main'); print('WORD_IMPORT_OK')"

# 檢查 PPT 腳本可載入（需帶 PYTHONPATH）
$env:PYTHONPATH="C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main"
$env:MCP_TRANSPORT="stdio"
& "C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\.venv\Scripts\python.exe" -c "import runpy; runpy.run_path(r'C:\Users\user\mcp\Office-PowerPoint-MCP-Server-main\ppt_mcp_server.py'); print('PPT_SCRIPT_LOAD_OK')"
```

## 6. 「已停止」常見原因
如果 UI 顯示 MCP `已停止`，常見原因是：

- MCP 程式在 `stdio` 模式把一般文字印到 `stdout`。
- MCP 客戶端期望 `stdout` 是 JSON-RPC 封包，結果讀到雜訊後握手失敗。
- 典型錯誤訊息：
  - `stream did not contain valid UTF-8`
  - `serde error expected value at line 1 column 1`

## 7. 這次已做的 Word 修正（重點）
修正檔案：

```text
C:\Users\user\mcp\Office-Word-MCP-Server-main\word_document_server\main.py
```

修正內容：
1. 新增 `_safe_log()`，把啟動訊息寫到 `stderr`。
2. 將原本 `print(...)`（啟動/傳輸資訊）改成 `_safe_log(...)`。
3. 透過 `sys.stdout.reconfigure(...)` / `sys.stderr.reconfigure(...)` 強制 UTF-8（errors=replace）。

目的：
- 避免污染 MCP `stdout` JSON 通道。
- 降低非 UTF-8 輸出造成的握手中斷。

## 8. 若仍顯示已停止，下一步怎麼看
檢查 VS Code 日誌：

```text
C:\Users\user\AppData\Roaming\Code\logs\<時間戳>\window1\exthost\openai.chatgpt\Codex.log
```

快速搜尋關鍵字：

```powershell
rg -n "CodexMcpConnection|valid UTF-8|serde error|word-document-server|ppt-server|excel-server|pdf-reader" "C:\Users\user\AppData\Roaming\Code\logs"
```

## 9. 版本差異注意事項
- 某些外掛版本可能使用 `servers`，有些可能使用 `mcpServers`。
- 若你看不到任何 MCP，先確認目前外掛版本的鍵名要求，再調整 `mcp.json`。

## 10. 建議流程（實務）
1. 先確認路徑都存在。
2. 單獨用命令列啟動 server，確認不秒退。
3. 再交給 VS Code MCP 管理。
4. 若失敗，第一時間查 `Codex.log` 的握手錯誤。

