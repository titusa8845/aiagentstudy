# Office-Word-MCP-Server 從零安裝到驗證 (Windows + Codex)

這份文件給教師研習/一般使用者使用，目標是在 Windows 上從「下載」到「安裝依賴」到「在 Codex 啟用 MCP」再到「實測讀取 docx」完整走一次。

本文件以 `GongRzhe/Office-Word-MCP-Server`（本機 MCP 名稱通常叫 `word-document-server`）為例。

---

## 1) 先備環境

1. 安裝 Python（建議 3.11+），並確認：

```powershell
python --version
python -m pip --version
```

2. （可選）安裝 Git；沒有 Git 也可以用 zip 下載。

```powershell
git --version
```

> 備註：這個 MCP server 處理的是 `.docx` 檔案（Office Open XML），通常不需要安裝 Microsoft Word 才能使用。

---

## 2) 下載原始碼

以下兩種方式擇一。

### 2.1 用 Git 下載（推薦）

```powershell
cd C:\Users\<你的使用者名稱>\Desktop
git clone https://github.com/GongRzhe/Office-Word-MCP-Server.git
cd Office-Word-MCP-Server
```

### 2.2 無 Git：用 zip 下載並解壓

```powershell
cd C:\Users\<你的使用者名稱>\Desktop

$zip = "$PWD\\Office-Word-MCP-Server-main.zip"
curl.exe -L -o $zip "https://github.com/GongRzhe/Office-Word-MCP-Server/archive/refs/heads/main.zip"
Expand-Archive -LiteralPath $zip -DestinationPath $PWD -Force

# 解壓後資料夾通常是 Office-Word-MCP-Server-main
cd .\\Office-Word-MCP-Server-main
```

---

## 3) 安裝方式（兩種擇一）

你要選「安裝到使用者環境（最簡單）」或「用 venv（最乾淨、可攜）」。

### 3A) 安裝到使用者環境（最簡單）

在 repo 根目錄執行：

```powershell
python -m pip install --user --upgrade pip
python -m pip install --user .
```

完成後，應該可以用這個指令啟動 server：

```powershell
python -m word_document_server.main
```

### 3B) 使用虛擬環境 venv（推薦給研習/教室環境）

在 repo 根目錄執行：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip install .
```

完成後，啟動 server 用：

```powershell
.\.venv\Scripts\python.exe -m word_document_server.main
```

---

## 4) 在 Codex 設定 MCP server

Codex 設定檔位置：
- `C:\Users\<你的使用者名稱>\.codex\config.toml`

在檔案末端加入（以下以「安裝到使用者環境」為例；如果你用 venv，請把 `command` 改成 `.venv\\Scripts\\python.exe` 的完整路徑）：

```toml
[mcp_servers.word-document-server]
command = "python"
args = ["-m", "word_document_server.main"]

[mcp_servers.word-document-server.env]
MCP_TRANSPORT = "stdio"
```

加入後請重新啟動 Codex，讓它載入設定。

---

## 5) 驗證 1：用 Codex 列出 MCP servers

在 PowerShell 執行：

```powershell
codex.cmd mcp list
```

預期會看到 `word-document-server` 為 `enabled`。

---

## 6) 驗證 2：列出 tools（建議用「存成 .py 檔」避免編碼問題）

在任意資料夾建立 `test_word_mcp_list_tools.py`，內容如下：

```python
import asyncio
from fastmcp import Client

config = {
  "mcpServers": {
    "word-document-server": {
      "command": "python",
      "args": ["-m", "word_document_server.main"],
      "env": {"MCP_TRANSPORT": "stdio"},
    }
  }
}

async def main():
  client = Client(config, timeout=60, init_timeout=60)
  async with client:
    tools = await client.list_tools()
    print("TOOLS_COUNT", len(tools))
    for t in tools[:20]:
      print("TOOL", t.name)

asyncio.run(main())
```

執行：

```powershell
python .\\test_word_mcp_list_tools.py
```

---

## 7) 驗證 3：讀取 docx 文字內容

建立 `test_word_mcp_read_docx.py`（把 `DOCX` 換成你的 docx 絕對路徑）：

```python
import asyncio
from fastmcp import Client

DOCX = r"C:\path\to\your.docx"

config = {
  "mcpServers": {
    "word-document-server": {
      "command": "python",
      "args": ["-m", "word_document_server.main"],
      "env": {"MCP_TRANSPORT": "stdio"},
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
  client = Client(config, timeout=60, init_timeout=60)
  async with client:
    r = await client.call_tool("get_document_text", {"filename": DOCX})
    print(extract_text(r)[:800])

asyncio.run(main())
```

執行：

```powershell
python .\\test_word_mcp_read_docx.py
```

---

## 8) 常見問題排除

1. 中文路徑變成 `??`
   - 常見原因是用 PowerShell 管線 `@' ... '@ | python -`，中途文字編碼把路徑弄壞。
   - 解法：改成「存成 UTF-8 的 `.py` 檔」再 `python xxx.py` 執行（上面的驗證方式就是）。
2. Codex/沙箱環境出現 `WinError 5 存取被拒`（子行程/pipe 權限）
   - 解法：用一般 PowerShell/Terminal 在沙箱外執行驗證腳本，或把驗證移到非受限環境再跑。

