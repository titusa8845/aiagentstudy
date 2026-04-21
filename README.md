# MCP Server 安裝教學合集（Windows + 教師研習用）by小萬

> **For AI agents:** This repository contains step-by-step installation guides for local MCP (Model Context Protocol) servers on Windows. Each document is a self-contained setup guide targeting teachers and educators. All servers use `stdio` transport and are intended to be shared across multiple AI clients (Gemini CLI, OpenCode, Codex/VS Code). Documents are written in Traditional Chinese.

更新日期：2026-04-21

---

## 本倉庫包含哪些文件

| 檔案 | 說明 |
|------|------|
| [`Office-PowerPoint-MCP-Server_從零安裝到驗證.md`](./Office-PowerPoint-MCP-Server_從零安裝到驗證.md) | PowerPoint MCP Server 完整安裝流程（研習版，含驗證） |
| [`Office-PowerPoint-MCP-Server_本機安裝紀錄_2026-03-27.md`](./Office-PowerPoint-MCP-Server_本機安裝紀錄_2026-03-27.md) | PowerPoint MCP 實際機器安裝紀錄（路徑與指令存檔） |
| [`Office-Word-MCP-Server_從零安裝到驗證.md`](./Office-Word-MCP-Server_從零安裝到驗證.md) | Word MCP Server 完整安裝流程（含 .docx 讀寫驗證） |
| [`Gemini-OpenCode-共用Office-PowerPoint-MCP-Server.md`](./Gemini-OpenCode-共用Office-PowerPoint-MCP-Server.md) | 讓 Gemini CLI + OpenCode + Codex 共用同一個 PPT MCP |
| [`Gemini-OpenCode-共用Office-Word-MCP-Server.md`](./Gemini-OpenCode-共用Office-Word-MCP-Server.md) | 讓 Gemini CLI + OpenCode + Codex 共用同一個 Word MCP |
| [`Gemini-OpenCode-共用Excel-MCP-Server.md`](./Gemini-OpenCode-共用Excel-MCP-Server.md) | Excel MCP Server 安裝與多客戶端共用設定 |
| [`Gemini-OpenCode-共用pdf-reader-mcp.md`](./Gemini-OpenCode-共用pdf-reader-mcp.md) | PDF Reader MCP 安裝與多客戶端共用設定 |
| [`VSCode_Codex_MCP_全域安裝與Word修正指南.md`](./VSCode_Codex_MCP_全域安裝與Word修正指南.md) | VS Code 全域 `mcp.json` 設定，含 word-document-server 停止問題修正 |

---

## 涵蓋的 MCP Servers

| MCP Server | 來源 | 技術 | 功能 |
|---|---|---|---|
| `Office-PowerPoint-MCP-Server` | [GongRzhe/Office-PowerPoint-MCP-Server](https://github.com/GongRzhe/Office-PowerPoint-MCP-Server) | Python + venv | 讀寫 `.pptx` |
| `Office-Word-MCP-Server` | [GongRzhe/Office-Word-MCP-Server](https://github.com/GongRzhe/Office-Word-MCP-Server) | Python + venv | 讀寫 `.docx` |
| `excel-mcp-server` | [haris-musa/excel-mcp-server](https://github.com/haris-musa/excel-mcp-server) | Python (`pip install`) | 讀寫 `.xlsx` |
| `pdf-reader-mcp` | [@sylphx/pdf-reader-mcp](https://www.npmjs.com/package/@sylphx/pdf-reader-mcp) | Node.js (`npx`) | 讀取 PDF 內容 |

---

## 適用的 AI 客戶端

所有文件的設定步驟均涵蓋以下一種或多種客戶端：

- **Codex**（VS Code 外掛，全域設定檔：`C:\Users\<user>\AppData\Roaming\Code\User\mcp.json`）
- **Gemini CLI**（設定檔：`C:\Users\<user>\.gemini\settings.json`）
- **OpenCode**（設定檔：`C:\Users\<user>\.opencode\config.json`）

---

## 快速閱讀指南

**我是 AI，需要幫用戶安裝某個 MCP：**
1. 先確認用戶想裝哪個 server（PowerPoint / Word / Excel / PDF）
2. 讀對應的「從零安裝到驗證」或「共用設定」文件
3. 注意：Python 系 server 一律用 venv 隔離，路徑用英文避免亂碼
4. 確認用戶使用哪個 AI 客戶端，套用對應的 config 格式

**我是老師/用戶，第一次設定：**
1. 從「從零安裝到驗證」系列文件開始
2. 確認先備環境（Python 3.11+ 或 Node.js LTS）
3. 路徑建議使用 `C:\Users\<你的帳號>\mcp\` 作為統一存放位置
4. 完成後用 [`VSCode_Codex_MCP_全域安裝與Word修正指南.md`](./VSCode_Codex_MCP_全域安裝與Word修正指南.md) 做全域整合

---

## 共同注意事項

- **使用英文路徑**：中文路徑在 PowerShell pipeline 中容易造成編碼錯誤
- **Python 系 server 一律使用 venv**：避免污染全域環境，方便版本固定
- **Windows PowerShell ExecutionPolicy**：若 `npm.ps1`/`npx.ps1` 被封鎖，改用 `npm.cmd`/`npx.cmd`
- **Transport 全部為 stdio**：不需要開 port，不需要防火牆設定

---

## For AI Agents — Quick Reference

```
Repo purpose   : MCP server setup guides for teachers on Windows
Language       : Traditional Chinese (zh-TW)
Transport      : stdio (all servers)
Python servers : use venv at C:\Users\<user>\mcp\<server-name>\.venv
Node servers   : use npx.cmd (not npx.ps1)
Config targets : VS Code mcp.json / Gemini settings.json / OpenCode config.json
Key constraint : English-only paths to avoid PowerShell encoding issues
```
