# pdf-reader-mcp (SylphxAI) 安裝與驗證步驟 (Windows + Codex)

這份文件給教師研習用，目標是讓學員從「環境準備」到「Codex 設定 MCP」再到「實際讀 PDF 驗證」都能一次做完。

## 1. 先備環境

### 1.1 安裝 Node.js (含 npm / npx)

1. 安裝 Node.js (建議 LTS 版)。
2. 開啟 PowerShell，確認版本:

```powershell
node -v
npm -v
npx -v
```

若以上任一指令不存在，通常是 Node.js 沒安裝或 PATH 沒設好。重開終端機後再試一次。

### 1.2 (建議) 安裝 Python

這不是 pdf-reader-mcp 的必要條件，但如果你想用本文提供的「Python 驗證腳本」列出工具或測讀 PDF，才需要 Python。

```powershell
python --version
python -m pip --version
```

## 2. 在 Codex 設定 pdf-reader-mcp

Codex 會從使用者目錄下的設定檔讀取 MCP servers。

1. 打開設定檔:
   - `C:\Users\<你的使用者名稱>\.codex\config.toml`
2. 加入以下設定 (Windows 請用 `npx.cmd`):

```toml
[mcp_servers.pdf-reader]
command = "npx.cmd"
args = ["-y", "@sylphx/pdf-reader-mcp"]
```

3. 重新啟動 Codex (讓它重新載入設定)。

備註:
- `-y` 代表 npx 第一次下載套件時自動同意。
- 第一次啟動會下載 npm 套件，可能需要 10 秒到數分鐘，視網路而定。

## 3. 驗證 MCP server 能啟動 (列出工具)

這一步用 Python + FastMCP 的 Client 直接連到 stdio MCP server，確認 `pdf-reader-mcp` 真的跑得起來。

在 PowerShell 執行:

```powershell
@'
import asyncio
from fastmcp import Client

config = {
  "mcpServers": {
    "pdf-reader": {
      "command": "npx.cmd",
      "args": ["-y", "@sylphx/pdf-reader-mcp"]
    }
  }
}

async def main():
  client = Client(config, timeout=30, init_timeout=30)
  async with client:
    tools = await client.list_tools()
    print("TOOLS_COUNT", len(tools))
    for t in tools:
      print("TOOL", t.name)

asyncio.run(main())
'@ | python -
```

預期輸出:
- `TOOLS_COUNT 1`
- `TOOL read_pdf`

## 4. 驗證可以讀 PDF

準備一個 PDF 檔案路徑，例如:
- `C:\Users\<你的使用者名稱>\Desktop\試題\三年級國語期末評量考試卷.pdf`

在 PowerShell 執行 (只抽第 1 頁做快速驗證):

```powershell
$pdf = "C:\Users\<你的使用者名稱>\Desktop\試題\三年級國語期末評量考試卷.pdf"

@"
import asyncio, json
from fastmcp import Client

PDF = r"$pdf"
config = {
  "mcpServers": {
    "pdf-reader": {
      "command": "npx.cmd",
      "args": ["-y", "@sylphx/pdf-reader-mcp"]
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
    r = await client.call_tool("read_pdf", {
      "sources": [{"path": PDF, "pages": "1"}],
      "include_metadata": True,
      "include_page_count": True
    })
    print(extract_text(r)[:1500])

asyncio.run(main())
"@ | python -
```

看到回傳的 JSON 片段或第 1 頁文字，就代表成功。

## 5. 常見問題排除

1. npx 第一次下載很慢或失敗
   - 檢查學校網路是否擋 npm registry
   - 改用手機網路或請資訊同仁協助代理/白名單
2. 讀到的文字很亂
   - PDF 若是掃描影像，抽取文字品質會受限
   - 先用 `pages: "1"` 測試，再逐頁擴大
3. 在 Codex 裡看不到工具
   - 確認 `config.toml` 位置正確
   - 確認段落是 `[mcp_servers.pdf-reader]` 而不是拼錯
   - 重啟 Codex 再試

