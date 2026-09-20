# 從 Markdown 提取內嵌 Base64 圖片的開源工具調查

## 概述

許多 Markdown 文件因使用截圖工具或複製貼上，內嵌了大量 base64 編碼的圖片資料（data URI），導致檔案體積暴增且難以版本控制。本報告調查可用於**將 inline base64 圖片擷取為獨立圖檔，並以 URL/相對路徑取代**的開源工具與 Python 套件。

## 現有工具

### 1. `md-unbase`（npm 套件，2026-02 發布）

- **倉庫**：[Bo-Tao/md-unbase](https://github.com/Bo-Tao/md-unbase)[^mdunbase-repo]
- **安裝**：`npm install -g md-unbase` 或 `npx md-unbase <file.md>`[^mdunbase-npm]
- **授權**：MIT
- **特色**：
  - 支援 PNG、JPEG、GIF base64 圖片
  - 以圖片內容的 MD5 hash 前 8 字元命名檔案 → **自動去重**
  - 圖片存於與 Markdown 檔案相同目錄，並自動改寫參照
  - 支援 `--dry-run` 預覽模式[^mdunbase-dryrun]
- **Regex 實現**：`!\[([^\]]*)\]\(data:image\/(png|jpeg|jpg|gif);base64,([^)]+)\)`[^mdunbase-source]

### 2. `md-extract-images`（Python CLI，via `uv`）

- **倉庫**：[sjwestern/md-extract-images](https://github.com/sjwestern/md-extract-images)[^mdextract-repo]
- **安裝**：透過 `uv` 執行（未發布至 PyPI），Python 3.12+
- **授權**：MIT
- **特色**：
  - 建立以 Markdown 檔名命名的目錄（如 `mydoc/`）存放圖片
  - 輸出新檔案 `mydoc.extracted.md`，或透過 `--overwrite` 直接覆蓋原檔
  - 零依賴（僅使用標準函式庫）[^mdextract-implementation]
- **Regex 實現**：`data:image/(png|jpg|jpeg|gif);base64,([A-Za-z0-9+/=]+)`[^mdextract-core]

### 3. `zilan` 中的 `MarkdownImageUtil.extract_base64()`（Python 參考實現）

- **倉庫**：[lxcshine/zilan](https://github.com/lxcshine/zilan)[^zilan-repo]
- **方法**：`docreader/parser/markdown_parser.py` 中的 `extract_base64()`[^zilan-source]
- **特色**：
  - 使用 UUID 產生唯一檔名，避免衝突
  - 其 Regex 處理了更多邊界情況：`[^;]+` 處理含連字號的 MIME subtype（如 `x-emf`），alt text 使用非貪婪匹配
- **Regex 實現**：`!\[(.*?)\]\((?:data:image/([^;]+);base64,([^\)]+))\)`[^zilan-source]

### 4. Pandoc `--extract-media=`（通用方案）

- Pandoc 的 `--extract-media=DIR` 標記可從來源文件提取內嵌媒體[^pandoc-media]
- 主要設計用於二進位容器（docx、epub）或外部連結，非專為 base64 設計，但可透過 `pandoc --from markdown --to markdown --extract-media=./assets file.md` 達到類似效果

## 自建方案（無合適套件時的替代路徑）

由於目前**沒有成熟、廣泛採用的 PyPI 專用套件**，自行實作約需 20-30 行 Python，僅依賴標準函式庫 `re`、`base64`、`os`：

```python
import re, base64, os, uuid

def extract_base64_images(md_text, output_dir="assets"):
    os.makedirs(output_dir, exist_ok=True)
    pattern = r'!\[(.*?)\]\(data:image/([^;]+);base64,([^\)]+)\)'

    def replacer(match):
        alt, fmt, b64_data = match.groups()
        ext = fmt.split('+')[0]  # 處理如 svg+xml
        filename = f"{uuid.uuid4().hex[:8]}.{ext}"
        filepath = os.path.join(output_dir, filename)
        with open(filepath, "wb") as f:
            f.write(base64.b64decode(b64_data))
        return f"![{alt}]({filepath})"

    return re.sub(pattern, replacer, md_text)
```

## 相關專案

- **[Encephos/markdown-link-resolver](https://github.com/Encephos/markdown-link-resolver)**（58★，可 pip 安裝）— 反方向：將相對路徑的圖片解析為 inline base64，用於 RAG/LLM 場景[^link-resolver]
- **[rainstf/Base64ImageEmbedder](https://github.com/rainstf/Base64ImageEmbedder)** — 將圖片嵌入為 Markdown base64（反方向）[^embedder]
- **[jrb00013/pdf2text-studio](https://github.com/jrb00013/pdf2text-studio)** — PDF→Markdown 轉換，支援「base64 嵌入**或提取為獨立檔案**」選項[^pdf2text]

## 結論與建議

| 方案 | 語言 | 安裝方式 | 成熟度 | 推薦場景 |
|------|------|----------|--------|----------|
| `md-unbase` | Node.js | `npx md-unbase` | 新發布、有 npm | Node.js 生態使用者 |
| `md-extract-images` | Python 3.12+ | `uv run` | 新發布、零依賴 | Python 生態使用者 |
| 自建腳本 | Python 3+ | 直接執行 | 高度可控 | 需要自訂輸出規則/目錄結構 |

若僅需一次性轉換，`md-unbase`（`npx md-unbase file.md`）或 `md-extract-images` 即可滿足需求。若需要更細緻的控制（如自訂目錄、智慧命名、批次處理），參考上述實作編寫 30 行左右的 Python 腳本是最務實的選擇。

[^mdunbase-repo]: Bo-Tao. (2026). *md-unbase — Extract base64 images from Markdown files*. GitHub. Retrieved 2026-09-20, from https://github.com/Bo-Tao/md-unbase
[^mdunbase-npm]: Bo-Tao. (2026). *md-unbase v0.1.0*. npm Registry. Retrieved 2026-09-20, from https://registry.npmjs.org/md-unbase
[^mdunbase-dryrun]: md-unbase `src/parser.ts`. Retrieved 2026-09-20, from https://raw.githubusercontent.com/Bo-Tao/md-unbase/main/src/parser.ts
[^mdunbase-source]: md-unbase `src/extractor.ts`. Retrieved 2026-09-20, from https://raw.githubusercontent.com/Bo-Tao/md-unbase/main/src/extractor.ts
[^mdextract-repo]: sjwestern. (n.d.). *md-extract-images*. GitHub. Retrieved 2026-09-20, from https://github.com/sjwestern/md-extract-images
[^mdextract-implementation]: md-extract-images `pyproject.toml` — zero dependencies. Retrieved 2026-09-20, from https://raw.githubusercontent.com/sjwestern/md-extract-images/main/pyproject.toml
[^mdextract-core]: md-extract-images `src/md_extract_images/core.py`. Retrieved 2026-09-20, from https://raw.githubusercontent.com/sjwestern/md-extract-images/main/src/md_extract_images/core.py
[^zilan-repo]: lxcshine. (n.d.). *zilan*. GitHub. Retrieved 2026-09-20, from https://github.com/lxcshine/zilan
[^zilan-source]: zilan `docreader/parser/markdown_parser.py` — `MarkdownImageUtil.extract_base64()`. Retrieved 2026-09-20, from https://raw.githubusercontent.com/lxcshine/zilan/main/docreader/parser/markdown_parser.py
[^pandoc-media]: Pandoc. (n.d.). *Pandoc User's Guide — `--extract-media`*. Retrieved 2026-09-20, from https://pandoc.org/MANUAL.html#extracting-media
[^link-resolver]: Encephos. (n.d.). *markdown-link-resolver*. GitHub. Retrieved 2026-09-20, from https://github.com/Encephos/markdown-link-resolver
[^embedder]: rainstf. (n.d.). *Base64ImageEmbedder*. GitHub. Retrieved 2026-09-20, from https://github.com/rainstf/Base64ImageEmbedder
[^pdf2text]: jrb00013. (n.d.). *pdf2text-studio*. GitHub. Retrieved 2026-09-20, from https://github.com/jrb00013/pdf2text-studio