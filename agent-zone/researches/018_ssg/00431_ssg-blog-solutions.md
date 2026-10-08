# SSG（靜態網站生成器）FOSS 方案研究報告

## 概述

本報告針對「將 Markdown 檔案轉換為網站、支援標籤/分類功能、專注於部落格使用場景」的需求， 調查市面上主流的 FOSS（自由與開源軟體）靜態網站生成器。

## 主要候選方案

### 1. Hugo（Go 語言）

- **建置速度最快**：1,000 頁約 5 秒，單頁 <1ms
- **標籤系統最完善**：內建 Taxonomy 系統，標籤與分類是一級公民（first-class），於 config 定義後即可在前言（frontmatter）中設定，自動生成標籤頁與各標籤的 RSS
- **Markdown**：支援 Goldmark 解析器，無原生 MDX，但可用 shortcodes 嵌入複雜內容
- **部落格友善度**：極佳，超過 1,000 個主題，內建 RSS、Sitemap、多國語系 i18n、圖片處理、Sass
- **學習曲線**：中高——Go 模板語法較特殊，對 JavaScript 開發者不直觀
- **社群**：GitHub 約 89K stars，生態成熟活躍[^hugo]

### 2. Zola（Rust 語言）

- **速度接近 Hugo**：1,000 頁約 7 秒，單一二進位檔，無執行期依賴
- **標籤系統**：內建 Taxonomy，與 Hugo 一樣開箱即用，自動生成標籤與分類頁
- **Markdown**：CommonMark 規範 + GFM（表格、任務清單、刪除線、註腳），支援簡碼（shortcodes）
- **部落格友善度**：極佳，內建 Sass 編譯、搜尋索引、RSS、即時重新載入
- **學習曲線**：低——Tera 模板（Jinja2/Django 風格）對 Python 開發者非常熟悉。30 分鐘即可上線
- **社群**：約 13K stars，偏小但活躍，主題生態較 Hugo 少[^zola]

### 3. Astro（JavaScript / TypeScript）

- **2026 年新專案的預設選擇**：預設零 JavaScript，採用 Island 架構
- **標籤系統**：Content Collections API + 篩選，在前言中設定 tags 並產生標籤頁，但需手動設定
- **Markdown**：原⽣ MDX 支援、Content Collections 結構驗證、圖片最佳化
- **部落格友善度**：極佳，Starlight 文件主題、內容優先架構，Core Web Vitals 表現突出
- **學習曲線**：低——.astro 檔案類似 HTML+，可帶入 React/Vue/Svelte 元件
- **社群**：成長最快，約 62K stars，每週 npm 下載量 270 萬[^astro]

### 4. 11ty / Eleventy（JavaScript）

- **極簡主義者首選**：「內容 + 模板 = HTML」，不輸出任何 JavaScript
- **標籤系統**：Collections-based，透過前言與模板定義集合，靈活但需自行設定
- **Markdown**：可透過外掛支援 MDX，支援 10 種以上模板語言
- **部落格友善度**：極佳，零設定即可運作，乾淨的 HTML 輸出
- **學習曲線**：低——對 JavaScript 開發者友善，設定極簡
- **社群**：約 17K stars，正在成長中[^11ty]

### 5. Jekyll（Ruby）

- **祖父級 SSG**：原生 GitHub Pages 整合是其殺手級功能
- **標籤系統**：內建 categories 與 tags，可透過 Liquid 或外掛產生各標籤頁面
- **Markdown**：Kramdown 解析器，無原生 MDX
- **部落格友善度**：極佳——本質即為部落格引擎，永久連結、分類、標籤、分頁、RSS、主題一應俱全
- **學習曲線**：低——Liquid 模板簡單易懂，但 Ruby 依賴管理較麻煩
- **社群**：約 52K stars，但已無重大更新，生態逐漸轉移至 Eleventy 與 Astro[^jekyll]

### 6. Pelican（Python）

- **Python 開發者的 SSG**：支援 Markdown、reStructuredText、AsciiDoc
- **標籤系統**：內建 categories 與 tags，透過檔案的 metadata 設定，自動產生標籤與分類頁
- **Markdown**：支援多種格式，可透過外掛擴充
- **部落格友善度**：良好——時間軸文章、靜態頁面、多國語系、Atom/RSS、程式碼高亮、WordPress 匯入
- **學習曲線**：低——Jinja2 模板廣泛使用，`pelican-quickstart` 快速入門
- **社群**：約 13K stars，穩定但非快速成長[^pelican]

### 7. Hexo（JavaScript / Node.js）

- **專為部落格而生**：在中文開發者社群中非常流行
- **標籤系統**：內建 tags 與 categories，前言中直接設定
- **Markdown**：支援完整 GFM，包含多數 Octopress 外掛
- **部落格友善度**：極佳——一鍵部署到 GitHub Pages、Heroku 等
- **學習曲線**：極低——`npm install -g hexo-cli` 即可開始
- **社群**：約 42K stars，亞洲社群活躍[^hexo]

## 比較總結

| SSG | 語言 | 建置速度 | JS 體積 | Markdown | 標籤系統 | 部落格友善 | 學習曲線 | 社群活躍度 |
|---|---|---|---|---|---|---|---|---|
| **Hugo** | Go | 極快 (~5s/1K頁) | 0 KB | 良好 | ★★★★★ 內建 Taxonomy | ★★★★★ | 中高 | 🟢 活躍 |
| **Zola** | Rust | 極快 (~7s/1K頁) | 0 KB | 極佳 (CommonMark+GFM) | ★★★★★ 內建 Taxonomy | ★★★★★ | 低 | 🟡 小型活躍 |
| **Astro** | JS/TS | 快 (~45s/1K頁) | 0 KB (Islands) | 極佳 (原生 MDX) | ★★★★ Content Collections | ★★★★★ | 低 | 🟢 成長最快 |
| **11ty** | JS | 快 (~30s/1K頁) | 0 KB | 良好 (可外掛 MDX) | ★★★★ Collections-based | ★★★★ | 低 | 🟢 成長中 |
| **Jekyll** | Ruby | 慢 (~150s/1K頁) | 0 KB | 良好 (Kramdown) | ★★★★ 內建分類/標籤 | ★★★★★ | 低 | 🟡 穩定/趨緩 |
| **Pelican** | Python | 中等 | 0 KB | 良好 (多格式) | ★★★★ 內建分類/標籤 | ★★★★ | 低 | 🟡 穩定 |
| **Hexo** | JS/Node | 快 | 0 KB | 良好 (GFM) | ★★★★ 內建標籤/分類 | ★★★★★ | 極低 | 🟢 中型活躍 |

## 建議

根據不同偏好，對應推薦如下：

| 偏好情境 | 推薦方案 |
|---|---|
| 追求**最快建置 + 最完善標籤系統** | Hugo 或 Zola |
| 想要**現代 JS 生態 + 零 JS 開銷** | Astro（或 11ty 追求極簡） |
| 需要**GitHub Pages 免費託管免設定 CI** | Jekyll |
| **Python 開發者**熟悉 Jinja2 模板 | Pelican |
| 想要**單一二進位檔無依賴地獄** | Hugo（Go）或 Zola（Rust） |
| 最簡單的 **Markdown 轉部落格**體驗 | Hexo 或 Zola |
| 需要 **MDX + React 元件**在部落格中 | Astro |
| 極簡主義，**最大控制權最少預設** | 11ty |

---

[^hugo]: Hugo. (n.d.). *Hugo — Static Site Generator*. Retrieved 2026-10-03, from https://gohugo.io/
[^zola]: Zola. (n.d.). *Zola — Overview*. Retrieved 2026-10-03, from https://www.getzola.org/documentation/getting-started/overview/
[^astro]: Astro. (n.d.). *Astro — Build the Web*. Retrieved 2026-10-03, from https://astro.build/
[^11ty]: 11ty. (n.d.). *Eleventy — a simpler static site generator*. Retrieved 2026-10-03, from https://www.11ty.dev/
[^jekyll]: Jekyll. (n.d.). *Jekyll — Static Site Generator*. Retrieved 2026-10-03, from https://jekyllrb.com/
[^pelican]: Pelican. (n.d.). *Pelican — Static Site Generator*. Retrieved 2026-10-03, from https://getpelican.com/
[^hexo]: Hexo. (n.d.). *Hexo — A fast, simple & powerful blog framework*. Retrieved 2026-10-03, from https://hexo.io/