# Office-PowerPoint-MCP-Server 從零安裝到驗證 (Windows + Codex)

這份文件設計給教師研習使用，目標是讓學員在 Windows 上從「下載」到「安裝依賴」到「在 Codex 啟用 MCP」再到「實測產出 PPTX」完整走一次。

本文件以 `GongRzhe/Office-PowerPoint-MCP-Server` 為例。

## 0. 重點注意事項 (研習建議)

- 盡量使用「全英文路徑」安裝與放檔案，例如 `C:\mcp\...`。
  - 中文路徑通常也能用，但在 PowerShell 用管線（例如 `... | python -`）時容易遇到編碼問題。
- 這台環境的 PowerShell 可能禁止執行 `npm.ps1` / `npx.ps1`（ExecutionPolicy），所以若需要 Node 指令，請用 `npm.cmd` / `npx.cmd`。
- 這個 PowerPoint MCP Server 本身是 **Python** 為主，不需要 Node.js 才能跑。

## 1. 先備環境

### 1.1 安裝 Python

建議使用 Python 3.11 以上。安裝後在 PowerShell 確認:

```powershell
python --version
python -m pip --version
```

### 1.2 (可選) 安裝 Git

如果沒有 Git，也可以用本文的 zip 下載法完成。

```powershell
git --version
```

## 2. 下載原始碼

以下兩種方式擇一。

### 2.1 用 Git 下載 (推薦)

```powershell
cd C:\Users\<你的使用者名稱>\Desktop
git clone https://github.com/GongRzhe/Office-PowerPoint-MCP-Server.git
cd Office-PowerPoint-MCP-Server
```

### 2.2 無 Git：用 zip 下載並解壓

```powershell
cd C:\Users\<你的使用者名稱>\Desktop

$zip = "$PWD\\Office-PowerPoint-MCP-Server-main.zip"
curl.exe -L -o $zip "https://github.com/GongRzhe/Office-PowerPoint-MCP-Server/archive/refs/heads/main.zip"

Expand-Archive -LiteralPath $zip -DestinationPath $PWD -Force

# 解壓後資料夾通常是 Office-PowerPoint-MCP-Server-main
cd .\\Office-PowerPoint-MCP-Server-main
```

## 3. 建立虛擬環境並安裝依賴

在 repo 根目錄執行:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## 4. 在 Codex 設定 MCP server

Codex 設定檔位置:
- `C:\Users\<你的使用者名稱>\.codex\config.toml`

在檔案末端加入一段（請把路徑改成你實際的 repo 位置）:

```toml
[mcp_servers.ppt-server]
command = "C:\\path\\to\\Office-PowerPoint-MCP-Server-main\\.venv\\Scripts\\python.exe"
args = ["C:\\path\\to\\Office-PowerPoint-MCP-Server-main\\ppt_mcp_server.py"]

[mcp_servers.ppt-server.env]
PYTHONPATH = "C:\\path\\to\\Office-PowerPoint-MCP-Server-main"
MCP_TRANSPORT = "stdio"
```

設定完成後，重新啟動 Codex，讓它載入新的 MCP server。

## 5. 驗證 1：列出 tools

建立一個 Python 測試檔（避免 PowerShell 管線把中文路徑搞亂）:

```powershell
notepad .\\test_ppt_mcp_client.py
```

貼上以下內容並存檔（把 `REPO` 換成你的 repo 絕對路徑；建議用全英文路徑）:

```python
import asyncio
from fastmcp import Client

REPO = r"C:\path\to\Office-PowerPoint-MCP-Server-main"
PY = REPO + r"\.venv\Scripts\python.exe"
SCRIPT = REPO + r"\ppt_mcp_server.py"

config = {
  "mcpServers": {
    "ppt-server": {
      "command": PY,
      "args": [SCRIPT],
      "env": {"PYTHONPATH": REPO, "MCP_TRANSPORT": "stdio"},
    }
  }
}

async def main():
  client = Client(config, timeout=60, init_timeout=60)
  async with client:
    tools = await client.list_tools()
    print("TOOLS_COUNT", len(tools))
    for t in tools:
      print("TOOL", t.name)

asyncio.run(main())
```

執行:

```powershell
python .\\test_ppt_mcp_client.py
```

預期會看到多個工具，例如:
- `create_presentation`
- `open_presentation`
- `save_presentation`
- `add_slide`
- `extract_presentation_text`

## 6. 驗證 2：建立並輸出 PPTX

建立檔案 `test_ppt_mcp_e2e.py`，貼上:

```python
import asyncio
from fastmcp import Client

REPO = r"C:\path\to\Office-PowerPoint-MCP-Server-main"
PY = REPO + r"\.venv\Scripts\python.exe"
SCRIPT = REPO + r"\ppt_mcp_server.py"
OUT = r"C:\Users\<你的使用者名稱>\Desktop\ppt_mcp_test.pptx"

config = {
  "mcpServers": {
    "ppt-server": {
      "command": PY,
      "args": [SCRIPT],
      "env": {"PYTHONPATH": REPO, "MCP_TRANSPORT": "stdio"},
    }
  }
}

def extract_text(result):
  parts = []
  for item in getattr(result, "content", []) or []:
    txt = getattr(item, "text", None)
    if txt:
      parts.append(txt)
  return "\n".join(parts)

async def main():
  client = Client(config, timeout=90, init_timeout=90)
  async with client:
    r1 = await client.call_tool("create_presentation", {})
    print("CREATE:", extract_text(r1)[:300])

    r2 = await client.call_tool("save_presentation", {"file_path": OUT})
    print("SAVE:", extract_text(r2)[:300])

    r3 = await client.call_tool("open_presentation", {"file_path": OUT})
    print("OPEN:", extract_text(r3)[:300])

    r4 = await client.call_tool("get_presentation_info", {})
    print("INFO:", extract_text(r4)[:800])

asyncio.run(main())
```

執行:

```powershell
python .\\test_ppt_mcp_e2e.py
```

最後桌面應該會出現 `ppt_mcp_test.pptx`。

## 7. 常見問題排除

1. 啟動失敗: `WinError 2 系統找不到指定的檔案`
   - 檢查 `command` 指到的 `.venv\Scripts\python.exe` 是否存在
   - 檢查 `args` 指到的 `ppt_mcp_server.py` 是否存在
   - 建議把 repo 放到全英文路徑 (例如 `C:\mcp\Office-PowerPoint-MCP-Server`)
2. 工具回覆說找不到檔案
   - 請用絕對路徑，避免工作目錄不同導致相對路徑失效
3. 用 `... | python -` 測試時中文路徑變亂碼
   - 改用「存成 .py 檔」再 `python xxx.py` 執行，避免 PowerShell 管線編碼干擾

