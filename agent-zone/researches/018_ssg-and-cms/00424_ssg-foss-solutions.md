# SSG（靜態網站生成器）FOSS 解決方案研究

> 焦點：將 Markdown 轉換為網站的自由開源靜態網站生成器。

**撰寫日期**：2026-10-01

## 目錄

- [一、何謂靜態網站生成器（SSG）](#一何謂靜態網站生成器ssg)
- [二、熱門通用型 SSG](#二熱門通用型-ssg)
- [三、文件/文檔專用型 SSG](#三文件文檔專用型-ssg)
- [四、高效能輕量型 SSG](#四高效能輕量型-ssg)
- [五、較小眾/特殊用途 SSG](#五較小眾特殊用途-ssg)
- [六、快速對照表](#六快速對照表)
- [七、選擇建議](#七選擇建議)
- [參考資料](#參考資料)

---

## 一、何謂靜態網站生成器（SSG）

靜態網站生成器（Static Site Generator）是一種工具，它將 Markdown 等標記式語言寫作的內容，結合模板與主題，預先建置（build）為純 HTML/CSS/JS 的靜態網站。與動態網站（如 WordPress）不同，SSG 在部署前就完成所有頁面生成，因此具有：

- **高效能**：無需伺服端處理，CDN 可直接服務。
- **安全性**：無資料庫、無伺服端程式，攻擊面極小。
- **版本控制友善**：所有內容為純文字檔，可納入 Git 管理。
- **低成本**：可免費託管於 GitHub Pages、Netlify、Cloudflare Pages 等平台。

本報告收錄 22 個 FOSS（自由開源軟體）SSG 解決方案，按定位分類介紹。[^jamstack]

---

## 二、熱門通用型 SSG

### 1. Hugo

- **焦點領域**：通用網站、部落格、企業/政府/教育/新聞/活動網站
- **主要特色**：
  - 極快建置速度（毫秒級，萬頁網站秒內建成）[^hugo]
  - 單一二進位檔，無需任何 runtime 依賴
  - 內建多語言支援
  - 內建資產管線（圖片處理、JS bundling、Sass/SCSS 處理、TailwindCSS 支援）
  - 強大分類與標籤系統
- **程式語言**：Go（模板語言為 Go 模板語法）
- **授權條款**：Apache 2.0[^hugo-license]
- **易用性**：中等偏難。使用現成主題很簡單，但自訂模板需學習 Go 模板語法。

### 2. Jekyll

- **焦點領域**：部落格、個人網站、專案網站
- **主要特色**：
  - GitHub Pages 原生支援（push 後自動建置，零 CI 設定）[^jekyll]
  - 18 年累積的大量主題與插件生態
  - Liquid 模板引擎
  - Blog-aware（文章、分類、永久連結皆為一級公民）
- **程式語言**：Ruby（模板使用 Liquid）
- **授權條款**：MIT[^jekyll-github]
- **易用性**：簡單（若熟悉 Ruby），但本地需 Ruby 環境；大型網站建置變慢；專案已趨於停滯。

### 3. 11ty (Eleventy)

- **焦點領域**：通用網站、部落格、文件
- **主要特色**：
  - 零預設設定即可開始[^11ty]
  - 支援多種模板語言（Liquid、Nunjucks、Handlebars、Mustache、EJS、Haml、Pug、HTML、Markdown、WebC 等）
  - 快速建置
  - 不注入多餘 markup，無 telemetry
  - 被 NASA、Google、Microsoft 等企業使用
- **程式語言**：JavaScript (Node.js)
- **授權條款**：MIT
- **易用性**：高。零配置入門，文檔優秀，社群活躍。

### 4. Astro

- **焦點領域**：內容驅動網站（行銷、部落格、電子商務、文件）
- **主要特色**：
  - Server-First 架構，預設零 JS 送出[^astro]
  - Islands 架構優化效能
  - 支援 React、Vue、Svelte、SolidJS 等 UI 框架
  - Content Collections 提供型別安全的 Markdown
  - 內建圖片優化、View Transitions
  - 多種部署配接器（Netlify、Vercel、AWS 等）
- **程式語言**：JavaScript/TypeScript（模板使用 Astro 元件語法，HTML-like）
- **授權條款**：MIT
- **易用性**：中。懂 HTML 即可開始，但進階功能需 JS/TypeScript 知識；版本迭代較快。

### 5. Next.js

- **焦點領域**：全端應用（靜態導出為選項，主為 SSR 框架）
- **主要特色**：
  - React 生態系[^nextjs]
  - ISR（Incremental Static Regeneration）
  - Server Actions
  - 多種渲染模式（SSR/SSG）
  - Vercel 原生整合
- **程式語言**：JavaScript/TypeScript（React 元件）
- **授權條款**：MIT
- **易用性**：中。React 開發者友好，但純靜態網站使用 Next.js 偏重量級。

---

## 三、文件/文檔專用型 SSG

### 6. MkDocs

- **焦點領域**：專案文件（純靜態文件網站）
- **主要特色**：
  - 簡單 YAML 設定[^mkdocs]
  - 內建 dev-server 即時預覽
  - 搜尋功能
  - 多種主題（含 readthedocs 風格）
- **程式語言**：Python（模板使用 Jinja2）
- **授權條款**：BSD 3-Clause
- **易用性**：非常高。一頁 YAML + Markdown 即可，最簡單的 SSG 之一。

### 7. Material for MkDocs

- **焦點領域**：專業文件網站（MkDocs 的主題/擴充套件）
- **主要特色**：
  - Material Design 美觀主題[^material-mkdocs]
  - 多裝置適配
  - 內建搜尋（無第三方服務）
  - 程式碼註解與複製
  - 社交卡片
  - 10,000+ 圖示與表情
  - 暗色模式
  - 可離線使用
- **程式語言**：Python（MkDocs 之上，同 Jinja2）
- **授權條款**：MIT（底層 MkDocs 為 BSD 3-Clause）
- **易用性**：非常高。在 MkDocs 基礎上追加極少設定即可啟用。

### 8. Docusaurus

- **焦點領域**：產品文件（含多版本/多語言）
- **主要特色**：
  - MDX（Markdown 中嵌入 React 元件）[^docusaurus]
  - 文件版本化
  - 多語言 i18n（內建）
  - Algolia 文件搜尋
  - 由 Meta 維護
- **程式語言**：JavaScript/TypeScript（React、MDX）
- **授權條款**：MIT
- **易用性**：中。簡單文件輕鬆上手，但需 Node.js 工具鏈；React 知識在進階自訂時必要。

### 9. VitePress

- **焦點領域**：文件網站
- **主要特色**：
  - 基於 Vite 建置（極快開發啟動與熱更新）[^vitepress]
  - Markdown 寫作
  - Vue 語法可直接在 Markdown 中使用
  - 靜態 HTML 快速初始載入
  - 客戶端路由快速導航
- **程式語言**：JavaScript（Vue 生態系）
- **授權條款**：MIT
- **易用性**：高。簡潔設定，Vite 開發體驗極佳。

---

## 四、高效能輕量型 SSG

### 10. Zola

- **焦點領域**：通用網站、部落格、知識庫、登陸頁
- **主要特色**：
  - 單一二進位檔（無任何依賴）[^zola]
  - 極快建置速度（平均 <1 秒）
  - 內建 Sass 編譯、語法高亮、目錄生成
  - 增強 Markdown（內部連結等）
- **程式語言**：Rust（模板使用 Tera，類似 Jinja2）
- **授權條款**：MIT
- **易用性**：高。單一 binary，零依賴，快速上手。

### 11. Hexo

- **焦點領域**：部落格
- **主要特色**：
  - 快速建置（Node.js）[^hexo]
  - 完整 GitHub Flavored Markdown 支援
  - 一指令部署到 GitHub Pages/Heroku 等
  - 豐富插件系統（支援 EJS、Pug、Nunjucks 等）
  - 大量主題（約 455 個）
- **程式語言**：JavaScript (Node.js)
- **授權條款**：MIT
- **易用性**：中。部落格用戶友善，但需要 Node.js 環境。

### 12. Pelican

- **焦點領域**：部落格、多語言網站
- **主要特色**：
  - 支援 reStructuredText 與 Markdown[^pelican]
  - 多語言文章發佈
  - Atom/RSS 產生
  - Pygments 語法高亮
  - WordPress/Dotclear/RSS 內容匯入
  - 快取增量重建
- **程式語言**：Python（模板使用 Jinja2）
- **授權條款**：AGPL 3.0
- **易用性**：中。Python 用戶友善，未接觸 Python 者需學習。

### 13. Nikola

- **焦點領域**：部落格、通用網站（「batteries included」哲學）
- **主要特色**：
  - 多種輸入格式（reStructuredText、Markdown、Jupyter Notebooks、HTML）[^nikola]
  - 增量重建（只重建更改頁面）
  - 多語言支援
  - 內建評論、標籤、分類、RSS/Atom
- **程式語言**：Python 3（模板使用 Mako 或 Jinja2）
- **授權條款**：MIT
- **易用性**：中。功能豐富但設定選項較多。

---

## 五、較小眾/特殊用途 SSG

### 14. Bridgetown

- **焦點領域**：靜態網站 + 全端應用混合
- **主要特色**：Ruby 生態系；靜態生成 + Roda 動態路由後端；多模板引擎（ERB、Serbea、Liquid）；元件化視圖層；現代前端建置（esbuild + PostCSS）[^bridgetown]
- **程式語言**：Ruby
- **授權條款**：MIT
- **易用性**：中。Ruby 開發者理想，但社群較小。

### 15. Lektor

- **焦點領域**：靜態 CMS（需要後台編輯的靜態網站）
- **主要特色**：可自訂的管理後台（瀏覽器編輯）；檔案式平面資料庫；依賴追蹤（僅重建變更頁面）；圖片工具（縮圖、EXIF）[^lektor]
- **程式語言**：Python（管理後台含 Node.js）
- **授權條款**：BSD 3-Clause
- **易用性**：中低。需學習其資料模型概念，但管理後台方便非技術使用者。

### 16. Cobalt

- **焦點領域**：通用靜態網站
- **主要特色**：簡單易用；Rust 撰寫（極快）；工作流程導向 CLI[^cobalt]
- **程式語言**：Rust
- **授權條款**：MIT
- **易用性**：中。文檔較簡略，社群小。

### 17. Publii

- **焦點領域**：桌面 GUI 為主的靜態 CMS（非技術使用者）
- **主要特色**：桌面應用程式（所見即所得編輯）；不需 CLI；一鍵發布到多種伺服器；主題商店[^publii]
- **程式語言**：JavaScript（Electron，模板使用 Handlebars）
- **授權條款**：GPL 3.0
- **易用性**：最高。完全 GUI，無需指令列。

### 18. mdBook

- **焦點領域**：文件網站（GitBook 替代）
- **主要特色**：Rust 撰寫；純 Markdown 輸入；可嵌入程式碼執行結果；多種輸出格式；Rust 社群廣泛使用（如 Rust 官方書）[^mdbook]
- **程式語言**：Rust（模板使用 Markdown）
- **授權條款**：MPL 2.0
- **易用性**：高。專注文件，單一用途，簡單配置。

### 19. Middleman

- **焦點領域**：通用網站、前端開發
- **主要特色**：Ruby 生態系；成熟穩定；支援多種模板引擎（ERB、Haml、Tilt）；豐富擴展[^middleman]
- **程式語言**：Ruby
- **授權條款**：MIT
- **易用性**：中。Ruby 開發者友善，但專案活性較低。

### 20. Metalsmith

- **焦點領域**：極簡、高度可定製的 SSG
- **主要特色**：全插件架構（一切皆 plugin）；極簡核心；最大靈活性[^metalsmith]
- **程式語言**：JavaScript (Node.js)
- **授權條款**：MIT
- **易用性**：中低。靈活但需自行組合插件。

### 21. Starlight

- **焦點領域**：文件網站（Astro 生態系中）
- **主要特色**：以 Astro 為基礎的文件主題；無障礙友善；高效能；搜尋、導航等開箱即用[^starlight]
- **程式語言**：JavaScript/TypeScript（Astro 生態）
- **授權條款**：MIT
- **易用性**：高。若已用 Astro，無痛擴充。

### 22. Sphinx

- **焦點領域**：Python 專案文件（經典工具）
- **主要特色**：reStructuredText 為主（亦支援 Markdown via MyST）；極度成熟；大量擴展（autodoc 等）；Python 社群標準[^sphinx]
- **程式語言**：Python（模板使用 Jinja2）
- **授權條款**：BSD 2-Clause
- **易用性**：中低。設定複雜，但 Python 開發者必備。

---

## 六、快速對照表

| 名稱 | 語言 | 授權 | 主要用途 | 易用性 |
|------|------|------|----------|--------|
| **Hugo** | Go | Apache 2.0 | 通用（速度最快） | ★★★☆ |
| **Jekyll** | Ruby | MIT | 部落格/GitHub Pages | ★★★★ |
| **11ty** | JS | MIT | 通用（最靈活） | ★★★★★ |
| **Astro** | JS/TS | MIT | 內容網站（少 JS） | ★★★★ |
| **Next.js** | JS/TS | MIT | 全端/靜態導出 | ★★★☆ |
| **MkDocs** | Python | BSD 3C | 文件（最簡單之一） | ★★★★★ |
| **Material-MkDocs** | Python | MIT | 專業文件 | ★★★★★ |
| **Docusaurus** | JS | MIT | 產品文件（版本化） | ★★★☆ |
| **VitePress** | JS | MIT | 文件（Vite 體驗） | ★★★★ |
| **Zola** | Rust | MIT | 通用（零依賴） | ★★★★ |
| **Hexo** | JS | MIT | 部落格 | ★★★☆ |
| **Pelican** | Python | AGPL 3.0 | 部落格/多語言 | ★★★☆ |
| **Nikola** | Python | MIT | 通用（batteries） | ★★★☆ |
| **Bridgetown** | Ruby | MIT | 靜態+動態混合 | ★★★☆ |
| **Lektor** | Python | BSD 3C | 靜態 CMS | ★★☆☆ |
| **Publii** | JS (Electron) | GPL 3.0 | GUI 靜態 CMS | ★★★★★ |
| **mdBook** | Rust | MPL 2.0 | 文件 | ★★★★ |
| **Cobalt** | Rust | MIT | 通用（輕量） | ★★★☆ |
| **Starlight** | JS/TS | MIT | 文件（Astro） | ★★★★ |
| **Metalsmith** | JS | MIT | 極簡可定制 | ★★☆☆ |
| **Sphinx** | Python | BSD 2C | Python 文件 | ★★☆☆ |
| **Middleman** | Ruby | MIT | 通用（成熟穩定） | ★★★☆ |

---

## 七、選擇建議

- **想要最快最輕量？** → Hugo 或 Zola
- **零成本 GitHub Pages 託管？** → Jekyll
- **最靈活、多模板語言、穩定迭代？** → 11ty (Eleventy)
- **文件網站最簡單？** → MkDocs + Material for MkDocs
- **多版本產品文件？** → Docusaurus
- **前端開發者、React 生態？** → Astro 或 Next.js
- **非技術使用者（GUI）？** → Publii
- **Python 開發者？** → MkDocs、Pelican、Nikola
- **Rust 愛好者？** → Zola、mdBook、Cobalt
- **純 Markdown → 靜態網站，最小配置** → mdBook、MkDocs、VitePress

---

## 參考資料

[^jamstack]: Jamstack.org. (n.d.). Jamstack generators. Retrieved 2026-10-01, from https://jamstack.org/generators/

[^hugo]: Hugo. (n.d.). Hugo — The world's fastest framework for building websites. Retrieved 2026-10-01, from https://gohugo.io/

[^hugo-license]: Hugo. (n.d.). Hugo license. Retrieved 2026-10-01, from https://gohugo.io/about/license/

[^jekyll]: Jekyll. (n.d.). Jekyll — Transform your plain text into static websites and blogs. Retrieved 2026-10-01, from https://jekyllrb.com/

[^jekyll-github]: GitHub — jekyll/jekyll. (n.d.). Retrieved 2026-10-01, from https://github.com/jekyll/jekyll

[^11ty]: 11ty. (n.d.). Eleventy — A simpler static site generator. Retrieved 2026-10-01, from https://www.11ty.dev/

[^astro]: Astro. (n.d.). Astro — The web framework for content-driven websites. Retrieved 2026-10-01, from https://astro.build/

[^nextjs]: Vercel. (n.d.). Next.js — The React framework for production. Retrieved 2026-10-01, from https://nextjs.org/

[^mkdocs]: MkDocs. (n.d.). MkDocs — Project documentation with Markdown. Retrieved 2026-10-01, from https://www.mkdocs.org/

[^material-mkdocs]: Squidfunk. (n.d.). Material for MkDocs — A Material Design theme for MkDocs. Retrieved 2026-10-01, from https://squidfunk.github.io/mkdocs-material/

[^docusaurus]: Meta. (n.d.). Docusaurus — Build documentation websites with Markdown & React. Retrieved 2026-10-01, from https://docusaurus.io/

[^vitepress]: VitePress. (n.d.). VitePress — Vite & Vue powered static site generator. Retrieved 2026-10-01, from https://vitepress.dev/

[^zola]: Zola. (n.d.). Zola — A fast static site generator in Rust. Retrieved 2026-10-01, from https://getzola.org/

[^hexo]: Hexo. (n.d.). Hexo — A fast, simple & powerful blog framework. Retrieved 2026-10-01, from https://hexo.io/

[^pelican]: Pelican. (n.d.). Pelican — A static site generator powered by Python. Retrieved 2026-10-01, from https://getpelican.com/

[^nikola]: Nikola. (n.d.). Nikola — A static site and blog generator. Retrieved 2026-10-01, from https://getnikola.com/

[^bridgetown]: Bridgetown. (n.d.). Bridgetown — A modern static site generator for the Ruby ecosystem. Retrieved 2026-10-01, from https://www.bridgetownrb.com/

[^lektor]: Lektor. (n.d.). Lektor — A static CMS with a beautiful admin panel. Retrieved 2026-10-01, from https://www.getlektor.com/

[^cobalt]: Cobalt. (n.d.). Cobalt — A static site generator written in Rust. Retrieved 2026-10-01, from https://cobalt-org.github.io/

[^publii]: Publii. (n.d.). Publii — The static CMS with a graphical interface. Retrieved 2026-10-01, from https://getpublii.com/

[^mdbook]: mdBook. (n.d.). mdBook — A utility to create books from Markdown files. Retrieved 2026-10-01, from https://github.com/rust-lang/mdBook

[^middleman]: Middleman. (n.d.). Middleman — A static site generator using Ruby. Retrieved 2026-10-01, from https://middlemanapp.com/

[^metalsmith]: Metalsmith. (n.d.). Metalsmith — An extremely simple, pluggable static site generator. Retrieved 2026-10-01, from https://metalsmith.io/

[^starlight]: Astro. (n.d.). Starlight — Build documentation websites with Astro. Retrieved 2026-10-01, from https://starlight.astro.build/

[^sphinx]: Sphinx. (n.d.). Sphinx — Python documentation generator. Retrieved 2026-10-01, from https://www.sphinx-doc.org/