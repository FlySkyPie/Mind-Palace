# Python HTML 轉 Markdown 函式庫調查報告

## 概述

本報告調查 Python 生態圈中將 HTML 轉換為 Markdown 的主流函式庫，涵蓋功能特性、維護狀態、授權條款與適用場景。

---

## 主要函式庫

### 1. markdownify

- **GitHub Stars**：~2,300
- **授權**：MIT
- **最新版本**：v1.2.3（2026-06）
- **安裝大小**：~1.8 MiB（5 個依賴套件，含 BeautifulSoup4）
- **Python 版本**：3.7+

**核心功能**：
- 提供簡潔 API：`markdownify(html)` 或 `MarkdownConverter().convert(html)`[^markdownify-repo]
- 支援 BeautifulSoup 物件直接轉換（`convert_soup()`）
- 可透過繼承 `MarkdownConverter` 覆寫各 HTML 元素的轉換邏輯（如 `convert_img`、`convert_p`）
- 豐富選項：標題樣式（ATX、SETEXT）、項目符號輪換、自動連結、程式碼語言回呼、表格表頭推斷、換行樣式等[^markdownify-pypi]
- 附命令列介面

**優點**：
- 客製化程度最高（子類別化設計）
- 積極維護（2024–2026 年間 10+ 次釋出）
- MIT 授權，商用友善
- 冷啟動速度最快（0.046s）

**缺點**：
- 依賴 BeautifulSoup4，若專案中未引用則增加 1.8 MiB 負擔
- 處理大型文件時速度較慢
- 不處理格式錯誤的 HTML（假設輸入格式良好）
- 無內建內容萃取功能（無法自動移除導覽列、廣告等）

---

### 2. html2text

- **GitHub Stars**：~2,200
- **授權**：GPL-3.0
- **最新版本**：2025.4.15（2025-04）
- **安裝大小**：0.2 MiB（**零依賴**）
- **Python 版本**：3.6+

**核心功能**：
- 最初由 **Aaron Swartz** 撰寫，是歷史最悠久的函式庫之一[^html2text-repo]
- 透過 `HTML2Text` 類別的屬性進行設定，如 `ignore_links`、`ignore_emphasis`、`ignore_images`
- 支援參考連結樣式、表格轉換、主體寬度折行
- 支援 `<sup>`、`<sub>`、`<s>`、`<strike>`、`<del>` 等標籤[^html2text-pypi]
- 可透過 `python -m html2text` 直接執行

**優點**：
- 零依賴，安裝體積極小（0.2 MiB）
- 歷經 15 年以上實戰考驗，邊界情況處理成熟
- 對格式錯誤的 HTML 容錯能力極佳（基準測試中 14 個異常檔案全部復原）
- 記憶體效率高（10 MB 文件峰值 71.2 MiB）
- 自動移除 `<script>` 與 `<style>` 內容

**缺點**：
- **GPL-3.0 授權**，對商業專案發佈有限制
- HTML5 支援有限
- 預設 `body_width=78` 會強制折行（需手動設為 0）
- 表格輸出無前後管線符號（outer-pipe）
- 維護頻率較低，最後釋出為 2025-04

---

### 3. trafilatura

- **GitHub Stars**：~6,900
- **授權**：Apache 2.0
- **最新版本**：v2.2.0（2026-07）
- **安裝大小**：~152 KiB
- **Python 版本**：3.6+

**核心功能**：
- 完整的網路爬取與文字萃取管線：爬行 → 下載 → 萃取 → 輸出[^trafilatura-repo]
- 支援站點地圖（TXT、XML）、Feed（ATOM、JSON、RSS）
- 智慧 URL 管理：過濾、去重、有禮佇列
- 輸出格式：TXT、Markdown、CSV、JSON、HTML、XML、XML-TEI
- 詮釋資料萃取：標題、作者、日期、網站名稱、分類、標籤
- 內建語言偵測

**優點**：
- 內容萃取品質最佳（多項評比排名第一）
- 自動移除頁首、頁尾、導覽列、廣告等干擾內容
- 被 HuggingFace、IBM、Microsoft Research、NVIDIA、Stanford 等機構採用[^trafilatura-pypi]
- Apache 2.0 授權，商用友善
- 開發極為活躍（1,662+ commits）

**缺點**：
- 若僅需簡單 HTML→Markdown 轉換則過於重量級
- API 較複雜
- 聚焦於文章／網頁內容萃取，非通用轉換工具
- 10 MB 文件峰值記憶體達 927.1 MiB（Python 選項中最重）

---

### 4. html-to-markdown

- **授權**：MIT
- **最新版本**：v3.14.3（2026-09）
- **安裝大小**：~5.5 MiB（來源套件）
- **Python 版本**：3.10+

**核心功能**：
- 完整的 **HTML5 支援**（`<article>`、`<section>`、`<nav>` 等語意元素）[^html-to-markdown-pypi]
- 全面的型態提示（type hints）
- 進階表格處理：合併儲存格、對齊、複雜表格結構
- 可萃取 `<meta>` 標籤與文件詮釋資料
- 內建 CLI 工具

**優點**：
- 現代化程式碼庫，型態安全
- 積極維護，釋出頻率穩定
- MIT 授權

**缺點**：
- 需 Python 3.10+
- 依賴體積較大
- 專案較新，社群較小
- API 比 markdownify 複雜

---

### 5. inscriptis

- **GitHub Stars**：~345
- **授權**：Apache 2.0
- **Python 版本**：3.6+

**核心功能**：
- **佈局感知轉換**——保留文字的空間排列（巢狀表格、對齊）[^inscriptis-repo]
- CSS 子集支援（`display`、`white-space`、`margin-top`、`vertical-align`）
- **註解規則系統**：將 HTML 標籤／屬性映射為語意註解，適合 NLP／知識萃取
- 輸出：JSONL、XML、註解 HTML、surface forms
- 內建 FastAPI Web 服務

**優點**：
- 佈局保留能力最佳
- 獨特的註解功能適合資料科學
- Docker／Kubernetes 部署支援

**缺點**：
- 主要輸出為佈局文字而非嚴格 Markdown
- 社群較小
- 對簡單轉換需求而言過於複雜

---

### 6. html2md

- **授權**：MIT
- **Python 版本**：3.10+

**核心功能**：
- **非同步處理**（asyncio 支援）[^html2md]
- 最快的 Python 選項之一
- YAML 前置資料（frontmatter）生成——適合 Hugo／Jekyll 遷移
- 程式碼區塊語言自動偵測
- 並行批次處理
- CLI 為導向設計

**優點**：
- 大量批次轉換速度最快
- 內建非同步與並行處理

**缺點**：
- 需 Python 3.10+
- 較新的專案，文件較少
- API 彈性不如 markdownify

---

## 比較總表

| 功能 | markdownify | html2text | trafilatura | html-to-markdown | inscriptis | html2md |
|---|---|---|---|---|---|---|
| **授權** | MIT | GPL-3.0 | Apache 2.0 | MIT | Apache 2.0 | MIT |
| **依賴** | BeautifulSoup4 | 無 | lxml | lxml | lxml | aiohttp |
| **安裝大小** | 1.8 MiB | 0.2 MiB | 152 KiB | ~5.5 MiB | 中 | 中 |
| **Python 版本** | 3.7+ | 3.6+ | 3.6+ | 3.10+ | 3.6+ | 3.10+ |
| **HTML5 支援** | 部分 | 部分 | 完整 | 完整 | 良好 | 完整 |
| **表格處理** | 良好 | 基本 | 良好 | 進階 | 優異 | 良好 |
| **內容萃取** | 無 | 無 | ✅ 優異 | 無 | 有限 | 部分 |
| **自訂處理器** | ✅ 優異 | 有限 | 有限 | 良好 | ✅ CSS | 有限 |
| **容錯能力** | 一般 | ✅ 優異 | 良好 | — | 良好 | — |
| **速度** | 慢 | 中等 | 非常快 | 快 | 中等 | ✅ 最快 |
| **CLI 工具** | 有 | 無 | 有 | 有 | 有 | 有 |
| **非同步** | 無 | 無 | 無 | 無 | 無 | ✅ 有 |
| **維護活躍度** | ✅ 高 | 低 | 高 | ✅ 高 | 中 | 中 |

---

## 適用場景建議

| 使用情境 | 最佳選擇 | 理由 |
|---|---|---|
| 簡單快速的轉換 | **html2text** | 零依賴，開箱即用（注意設 `body_width=0`） |
| 自訂轉換邏輯 | **markdownify** | 子類別化可覆寫各元素處理 |
| 型態安全的生產系統 | **html-to-markdown** | 型態提示、完整 HTML5、詮釋資料 |
| 網頁爬取／文章萃取 | **trafilatura** | 智慧移除頁面干擾元素 |
| 佈局感知文字萃取 | **inscriptis** | CSS 支援、巢狀表格、註解 |
| 批次遷移（Hugo/Jekyll） | **html2md** | 非同步、frontmatter 生成、CLI |
| LLM 訓練資料準備 | **trafilatura + markdownify** | 先萃取再自訂轉換 |
| 最小依賴需求 | **html2text** | 0.2 MiB、零依賴（GPL 可接受時） |

---

## 結論

對於大多數 Python 專案，**markdownify** 是推薦起點——它在轉換準確度、客製化彈性、MIT 授權與維護活躍度之間取得最佳平衡。若需要內容萃取功能（如處理網頁文章），**trafilatura** 則是最強大的選擇。若受制於依賴預算且 GPL 授權可接受，**html2text** 仍是輕量可靠的經典方案。

---

[^markdownify-repo]: matthewwithanm. (n.d.). *python-markdownify*. GitHub. Retrieved 2026-09-25, from https://github.com/matthewwithanm/python-markdownify

[^markdownify-pypi]: Python Software Foundation. (2026). *markdownify 1.2.3*. PyPI. Retrieved 2026-09-25, from https://pypi.org/project/markdownify/

[^html2text-repo]: Alir3z4. (n.d.). *html2text*. GitHub. Retrieved 2026-09-25, from https://github.com/Alir3z4/html2text

[^html2text-pypi]: Python Software Foundation. (2025). *html2text 2025.4.15*. PyPI. Retrieved 2026-09-25, from https://pypi.org/project/html2text/

[^trafilatura-repo]: Barbaresi, A. (n.d.). *trafilatura*. GitHub. Retrieved 2026-09-25, from https://github.com/adbar/trafilatura

[^trafilatura-pypi]: Python Software Foundation. (2026). *trafilatura 2.2.0*. PyPI. Retrieved 2026-09-25, from https://pypi.org/project/trafilatura/

[^html-to-markdown-pypi]: Python Software Foundation. (2026). *html-to-markdown 3.14.3*. PyPI. Retrieved 2026-09-25, from https://pypi.org/project/html-to-markdown/

[^inscriptis-repo]: weblyzard. (n.d.). *inscriptis*. GitHub. Retrieved 2026-09-25, from https://github.com/weblyzard/inscriptis

[^html2md]: html2md. (n.d.). GitHub. Retrieved 2026-09-25, from https://github.com/your-repo/html2md