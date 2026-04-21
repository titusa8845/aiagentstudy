# Office-PowerPoint-MCP-Server 本機安裝紀錄 (2026-03-27)

這份文件記錄「這台機器」實際完成的安裝步驟與驗證結果，方便回頭查設定與路徑。

## 1. 下載與解壓

- 下載 zip:
  - `C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main.zip`
- 解壓後 repo 位置:
  - `C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main`

使用的下載指令:

```powershell
curl.exe -L -o C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main.zip "https://github.com/GongRzhe/Office-PowerPoint-MCP-Server/archive/refs/heads/main.zip"
Expand-Archive -LiteralPath C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main.zip -DestinationPath C:\Users\JFPS\Desktop\試題 -Force
```

## 2. 建立 venv 與安裝依賴

venv 位置:
- `C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main\.venv`

安裝指令:

```powershell
cd C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## 3. Codex MCP 設定

設定檔:
- `C:\Users\JFPS\.codex\config.toml`

新增項目:

```toml
[mcp_servers.ppt-server]
command = "C:\\Users\\JFPS\\Desktop\\試題\\Office-PowerPoint-MCP-Server-main\\.venv\\Scripts\\python.exe"
args = ["C:\\Users\\JFPS\\Desktop\\試題\\Office-PowerPoint-MCP-Server-main\\ppt_mcp_server.py"]

[mcp_servers.ppt-server.env]
PYTHONPATH = "C:\\Users\\JFPS\\Desktop\\試題\\Office-PowerPoint-MCP-Server-main"
MCP_TRANSPORT = "stdio"
```

## 4. 工具列舉驗證

使用腳本:
- `C:\Users\JFPS\Desktop\試題\test_ppt_mcp_client.py`

結果:
- `TOOLS_COUNT 37`
- 例: `create_presentation`, `open_presentation`, `save_presentation`, `add_slide` 等

## 5. E2E 產生 PPTX 驗證

使用腳本:
- `C:\Users\JFPS\Desktop\試題\test_ppt_mcp_e2e3.py`

輸出檔:
- `C:\Users\JFPS\Desktop\ppt_mcp_test.pptx`

流程:
- `create_presentation` -> `save_presentation` -> `open_presentation` -> `get_presentation_info`

## 6. 小修正 (提升可用性)

為了讓 `save_presentation` / `get_presentation_info` 在建立或開啟簡報後「不用手動指定 presentation_id」也能直接用，
本機在 `ppt_mcp_server.py` 做了小修正：

- 檔案:
  - `C:\Users\JFPS\Desktop\試題\Office-PowerPoint-MCP-Server-main\ppt_mcp_server.py`
- 內容:
  - 當 `current_presentation_id` 尚未設定但 `presentations` 已有資料時，自動把最後載入的簡報設為 current。

