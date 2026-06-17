# MarkItDown MCP Server 設定教學（Windows + Docker）

> 使用 Microsoft 的 [markitdown](https://github.com/microsoft/markitdown) 搭配 Claude Desktop，將各種格式的檔案自動轉換成 Markdown。

---

## 環境需求

- Windows 10 / 11
- [Claude Desktop](https://claude.ai/download)
- [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
- Git

---

## Step 1：安裝 Docker Desktop

1. 前往 https://www.docker.com/products/docker-desktop/ 下載並安裝
2. 安裝完成後**重新啟動電腦**
3. 確認 Docker Desktop 已啟動（系統列出現 Docker 圖示）

驗證安裝：

```powershell
docker --version
```

---

## Step 2：Clone 原始碼並建立 Docker Image

```powershell
# Clone markitdown 原始碼（僅抓最新一版）
git clone https://github.com/microsoft/markitdown.git --depth 1 "C:\Users\<你的使用者名稱>\markitdown-src"

# 建立 Docker Image（需要幾分鐘，會下載 python:3.13-slim 基底）
docker build -t markitdown-mcp:latest "C:\Users\<你的使用者名稱>\markitdown-src\packages\markitdown-mcp"
```

驗證 Image 建立成功：

```powershell
docker images markitdown-mcp
```

---

## Step 3：設定 Claude Desktop

設定檔路徑：

```
C:\Users\<你的使用者名稱>\AppData\Roaming\Claude\claude_desktop_config.json
```

在 JSON 最外層加入 `mcpServers` 區塊：

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-v", "C:\\Users\\<你的使用者名稱>:/workdir",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

> **說明**：`-v C:\Users\<你的使用者名稱>:/workdir` 會將你的整個使用者資料夾掛載到容器內的 `/workdir`，讓 MCP server 可以存取本機檔案。

---

## Step 4：重新啟動 Claude Desktop

**完全關閉**再重開（右鍵系統列圖示 → Quit，不是只按 X）。

重開後在 Claude Desktop 的工具列看到 `markitdown` 工具即代表設定成功。

---

## 使用方式

### 轉換本機檔案

將檔案放在 `C:\Users\<你的使用者名稱>\` 底下任意資料夾，例如桌面：

```
幫我把 /workdir/Desktop/report.pdf 轉成 markdown 並摘要
```

路徑對應關係：

| 本機路徑 | 容器內路徑 |
|----------|------------|
| `C:\Users\user\Desktop\file.pdf` | `/workdir/Desktop/file.pdf` |
| `C:\Users\user\Documents\slide.pptx` | `/workdir/Documents/slide.pptx` |

### 轉換網路上的檔案

```
幫我把 https://example.com/document.pdf 轉成 markdown
```

---

## 支援的格式

| 類別 | 格式 |
|------|------|
| 文件 | PDF、Word (.docx)、PowerPoint (.pptx)、Excel (.xlsx) |
| 網頁 | HTML、XML |
| 圖片 | PNG、JPG、GIF、BMP、TIFF（使用 OCR） |
| 音訊 | MP3、WAV（使用語音轉文字） |
| 資料 | CSV、JSON |
| 其他 | ZIP（自動解壓並轉換內容）、EPub |

---

## 常見問題

**Q：Claude Desktop 沒有出現 markitdown 工具？**
- 確認 Docker Desktop 有在背景執行
- 確認 JSON 格式正確（可用 [jsonlint.com](https://jsonlint.com) 驗證）
- 確認有完全關閉再重開 Claude Desktop

**Q：找不到檔案？**
- 確認檔案位於 `C:\Users\<你的使用者名稱>\` 底下
- 路徑中的反斜線在 JSON 裡要寫成 `\\`

**Q：如何更新 Image？**
```powershell
cd "C:\Users\<你的使用者名稱>\markitdown-src"
git pull
docker build -t markitdown-mcp:latest packages\markitdown-mcp
```

---

## 相關連結

- [markitdown GitHub](https://github.com/microsoft/markitdown)
- [markitdown-mcp 套件](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp)
- [MCP 官方文件](https://modelcontextprotocol.io)
