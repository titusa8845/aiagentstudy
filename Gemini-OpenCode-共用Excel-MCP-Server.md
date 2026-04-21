# Gemini CLI + OpenCode 共用 Excel MCP（haris-musa/excel-mcp-server）

更新日期：2026-04-21

## 0) 先講結論（為什麼有人會說「不可用」）

`haris-musa/excel-mcp-server` 本體是 **Python 套件**，不是 npm 的 `@haris-musa/...` 套件。  
所以這個指令會失敗（E404）：

```powershell
npx.cmd -y @haris-musa/excel-mcp-server
```

可用做法是：
- `pip install excel-mcp-server`（本文件採用）
- 或 `uvx excel-mcp-server stdio`

---

## 1) 安裝策略（給老師的建議）

### 推薦策略：每台電腦用「獨立 venv + 固定版本」

優點：
- 不污染全域 Python
- 每台安裝結果一致，課堂較穩
- 路徑固定，方便批次設定 Codex/Gemini/OpenCode

本次固定版本：`excel-mcp-server==0.1.8`

---

## 2) 實際安裝步驟（Windows）

### 2.1 建資料夾與 venv

```powershell
mkdir C:\Users\user\mcp\excel-mcp-server
cd C:\Users\user\mcp\excel-mcp-server
py -m venv .venv
```

### 2.2 安裝套件

```powershell
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install excel-mcp-server==0.1.8
```

### 2.3 確認可啟動

```powershell
.\.venv\Scripts\excel-mcp-server.exe --help
```

應看到子命令包含：`stdio` / `sse` / `streamable-http`。

---

## 3) 三端共用設定

### 3.1 Codex

檔案：`C:\Users\user\.codex\config.toml`

```toml
[mcp_servers.excel-server]
command = "C:\\Users\\user\\mcp\\excel-mcp-server\\.venv\\Scripts\\excel-mcp-server.exe"
args = ["stdio"]
```

### 3.2 Gemini CLI

檔案：`C:\Users\user\.gemini\settings.json`

```json
"excel-server": {
  "command": "C:\\Users\\user\\mcp\\excel-mcp-server\\.venv\\Scripts\\excel-mcp-server.exe",
  "args": ["stdio"]
}
```

### 3.3 OpenCode

檔案：`C:\Users\user\.config\opencode\opencode.json`

```json
"excel-server": {
  "type": "local",
  "enabled": true,
  "command": [
    "C:\\Users\\user\\mcp\\excel-mcp-server\\.venv\\Scripts\\excel-mcp-server.exe",
    "stdio"
  ]
}
```

---

## 4) 驗證步驟

### 4.1 連線驗證

```powershell
codex.cmd mcp list
gemini.cmd -d mcp list
opencode.cmd mcp list
```

預期：三邊都看到 `excel-server` 且為 `connected/enabled`。

### 4.2 工具列出驗證

測試檔：`C:\Users\user\mcp\excel-mcp-server\test_excel_mcp_list_tools.py`

```powershell
C:\Users\user\mcp\excel-mcp-server\.venv\Scripts\python.exe C:\Users\user\mcp\excel-mcp-server\test_excel_mcp_list_tools.py
```

本次結果：`TOOLS_COUNT 25`（含 `create_workbook`、`write_data_to_excel`、`read_data_from_excel`）。

### 4.3 端到端讀寫驗證

測試檔：`C:\Users\user\mcp\excel-mcp-server\test_excel_mcp_e2e.py`

```powershell
C:\Users\user\mcp\excel-mcp-server\.venv\Scripts\python.exe C:\Users\user\mcp\excel-mcp-server\test_excel_mcp_e2e.py
```

本次結果：
- 成功建立 `excel_mcp_test.xlsx`
- 成功寫入資料
- 成功讀回 `A1:B3`

---

## 5) 課堂常見錯誤與排除

1. `E404 @haris-musa/excel-mcp-server`
- 原因：把 Python 套件當成 npm scoped package 安裝。
- 解法：改用 `pip install excel-mcp-server`（或 `uvx`）。

2. PowerShell 擋 `npm.ps1` / `npx.ps1`
- 解法：一律用 `npm.cmd` / `npx.cmd` / `codex.cmd` / `gemini.cmd` / `opencode.cmd`。

3. 中文路徑導致亂碼或啟動失敗
- 解法：安裝路徑改英文（如本文件的 `C:\Users\user\mcp\...`）。

---

## 6) 這台機器目前狀態

已完成：
- `excel-server` 安裝與設定
- 與 `codex` / `gemini cli` / `opencode` 共用成功
- 工具列出 + 讀寫 xlsx 驗證通過

