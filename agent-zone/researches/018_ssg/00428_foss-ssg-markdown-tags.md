# FOSS 靜態網站生成器 (SSG) 調查報告：Markdown 轉換與標籤功能支援

本報告調查當前主流的 FOSS（自由及開源軟體）靜態網站生成器（Static Site Generator, SSG），重點關注兩項需求：將 Markdown 檔案轉換為網站，以及支援標籤（tags）/分類（categories）功能。

## 調查範圍與方法

以網路搜尋與官方文件為主，調查截至 2026 年 10 月仍積極維護的 SSG 專案。

## 各 SSG 詳細分析

### Hugo

使用 Go 語言開發，單一二進位檔安裝，以極快的建置速度聞名——約 5 秒即可建置 1,000 頁[^hugo-speed]。GitHub 90K+ 星，每週釋出更新，維護狀態極佳。

**標籤/分類支援最強**。Hugo 擁有**使用者自訂分類系統**（taxonomies），不限於 `tags` 和 `categories`，可定義任意分類如 `actors`、`directors`、`genres` 等。在專案設定檔中配置後，於 front matter 以 YAML/TOML/JSON 陣列賦值。Hugo 會自動產生分類列表頁面（如 `/tags/`）與個別標籤頁面（如 `/tags/rust/`）。支援分類權重排序與每個分類的 RSS 訂閱[^hugo-taxo]。

**適用場景**：大型網站（萬頁以上）、文件入口站、重視建置速度者。

### Zola

使用 Rust 開發，單一二進位檔安裝，建置速度接近 Hugo（約 7 秒/千頁）[^zola-speed]。維護狀態良好。

**內建分類系統**，與 Hugo 類似。在 `zola.toml` 中以陣列配置分類物件，包含 `name`、`paginate_by`、`feed`、`render` 等選項。Front matter 中以 `[taxonomies]` TOML 區塊賦值。**大小寫不敏感**（`example` 與 `Example` 合併）。支援每個分類的 RSS、分頁、分類前綴選項[^zola-taxo]。

**適用場景**：想要 Hugo 速度但偏好更友善的模板（Tera，類似 Jinja2）的開發者。

### Jekyll

使用 Ruby 開發，是最早的 SSG 之一，GitHub Pages 原生支援無需 CI 設定即能部署[^jekyll-github]。建置速度較慢（約 150 秒/千頁）。

**內建標籤與分類支援**。Front matter 中以 `tags:` 定義標籤（陣列）。分類可在 front matter 中定義，也可**從目錄結構推斷**（如 `movies/horror/_posts/file.md` 自動歸類於 `movies` 和 `horror`）。以 Liquid 模板透過 `site.tags` 和 `site.categories` 存取。自動產生標籤/分類頁面需 `jekyll-archives` 外掛[^jekyll-tag]。

**適用場景**：GitHub Pages 使用者、現有 Jekyll 專案、Ruby 開發者。

### Eleventy / 11ty

使用 JavaScript/Node.js 開發，以零 JavaScript 輸出為預設，v3.0.0 於 2024 年 10 月釋出，Font Awesome 贊助支援[^11ty-v3]。

**使用標籤集合（tag-based collections）**。Front matter 中以 `tags` 鍵定義陣列，每個標籤自動建立一個具名集合（如 `collections.post`）。可透過 Collections API 的 `getFilteredByTag()` 過濾與分頁。若要區分「分類」和「標籤」，需在資料層自訂集合——11ty 提供的是基本元件而非意見型分類物件[^11ty-collections]。

**適用場景**：極簡主義者、想要完全控制權無框架魔術的開發者、部落格、小型文件站。

### Astro

使用 JavaScript/TypeScript 開發，採用島嶼架構（islands architecture），預設零 JavaScript 輸出，目前成長極為迅速。v5.x 已釋出，VC 資金支援[^astro-2026]。

**透過 Content Collections 管理標籤**。以 Zod schema 定義型別安全的 front matter，可將 `tags` 設為 `z.array(z.string())`。透過 `getCollection()` 查詢並手動過濾，或建立動態路由（如 `[tag].astro`）產生各標籤頁面。靈活度高但需較多手動設定，不如 Hugo/Zola 的自動化分類頁面[^astro-tag]。

**內建 MDX 支援**，可直接在 Markdown 中使用 React/Vue/Svelte 元件。

**適用場景**：行銷網站、部落格、文件站——被視為 2025/2026 年內容優先靜態站的新預設選擇。

### Pelican

使用 Python 開發，支援 Markdown、reStructuredText 和 AsciiDoc（透過外掛）。v4.12.0 於 2026 年 4 月釋出，維護狀態良好[^pelican-release]。

**內建標籤與分類支援**。Front matter 中以 `Tags: pelican, publishing`（逗號分隔）定義標籤。分類使用單一 `Category: Python` 欄位（一篇文章僅一個分類），也可從目錄結構推斷。Jinja2 模板中以 `article.tags` 和 `article.category` 存取。支援在內容中以 `{tag}tagname` 和 `{category}foobar` 連結至分類頁面[^pelican-docs]。

**適用場景**：Python 開發者、偏好 reStructuredText 者、現有 Pelican 專案。

### Hexo

使用 JavaScript/Node.js 開發，在東亞開發者社群特別受歡迎。

**內建標籤與分類支援**。分類支援**階層結構**——可定義巢狀分類（如 `[Sports, Baseball]`）。標籤為扁平結構，無階層。僅文章（posts）支援分類/標籤，頁面（pages）不支援[^hexo-front]。

**適用場景**：部落格為主的站點、偏好較簡單 Node.js 替代方案者。

### MkDocs

使用 Python 開發，專注於專案文件。

**標籤支援透過 Material for MkDocs 主題內建的外掛**（自 v8.2.0 起）。在頁面 front matter 定義標籤後，會產生標籤索引頁面並與搜尋整合。核心 MkDocs 本身無標籤功能，但 Material 主題的外掛已成事實標準[^mkdocs-tag]。

**適用場景**：專案文件站，非部落格用途。

### Next.js

使用 JavaScript/TypeScript (React) 開發，Vercel 支援，v15+ 已釋出。可靜態匯出（`output: 'export'`），但會攜帶 React Runtime（約 80KB）[^nextjs-export]。

**無內建標籤功能**，需自行實作。透過 `getStaticPaths()` 與 `getStaticProps()` 搭配 `gray-matter` 解析 front matter，從 `tags` 與 `categories` 陣列產生靜態頁面。社群有大量教學資源[^nextjs-blog]。

**適用場景**：同時需要靜態行銷頁面與互動應用區塊的專案、現有 React/Next.js 專案。

### SvelteKit

使用 JavaScript/Svelte 開發，Svelte 5 時代，v2.0+。可透過 `adapter-static` 靜態匯出，但攜帶 Svelte Runtime（約 30KB）。

**無內建標籤功能**，需自行實作。搭配 `mdsvex` 外掛處理 MDX。SvelteKit 的檔案路由使 `[tag].svelte` 動態頁面實作直接。靜態匯出能力良好[^sveltekit-static]。

**適用場景**：Svelte 開發者、輕量互動站、作品集。

## 比較總結

| SSG | 語言 | 標籤/分類 | 建置速度 (千頁) | 零 JS 輸出 | MDX |
|-----|------|-----------|----------------|-----------|-----|
| **Hugo** | Go | ✅ 使用者自訂分類 | ~5 秒 | ✅ | 有限 |
| **Zola** | Rust | ✅ 內建分類系統 | ~7 秒 | ✅ | 否 |
| **Jekyll** | Ruby | ✅ 內建 tags+cats | ~150 秒 | ✅ | 否 |
| **Eleventy** | JS | ✅ 標籤集合 (分類需 DIY) | ~30 秒 | ✅ | 外掛 |
| **Astro** | JS/TS | ✅ Content Collections | ~45 秒 | ✅ (島嶼架構) | ✅ 原生 |
| **Pelican** | Python | ✅ 內建 tags+category | 中等 | ✅ | 否 |
| **Hexo** | JS | ✅ 內建 (階層分類) | 中等 | ✅ | 否 |
| **MkDocs** | Python | ⚠️ 需 Material 主題 | 快 | ✅ | 否 |
| **Next.js** | JS/TS | ⚠️ DIY | ~90 秒 | ❌ (~80KB) | ✅ 原生 |
| **SvelteKit** | JS/Svelte | ⚠️ DIY | ~60 秒 | ❌ (~30KB) | 外掛 |

### 分類深度比較

| 功能 | Hugo | Zola | Jekyll | 11ty | Astro |
|------|------|------|--------|------|-------|
| 自訂分類 | ✅ 任何分類 | ✅ 任何分類 | ❌ tags/cats 僅 | ❌ 僅 tags | ✅ 在 schema 中 |
| 自動產生分頁 | ✅ 自動 | ✅ 自動 | ⚠️ 需外掛 | ✅ 每標籤集合 | ✅ 需手動 |
| 階層分類 | ✅ | ❌ | ✅ (從路徑) | ❌ | ✅ DIY |
| 每分類 RSS | ✅ | ✅ | ⚠️ 外掛 | ✅ | ✅ |
| 加權排序 | ✅ | ❌ | ❌ | ❌ | ❌ |
| 不分大小寫合併 | 可設定 | ✅ | ❌ | ❌ | 可設定 |

## 各場景推薦

- **「快速建立部落格」** → **Astro**（現代、Content Collections 型別安全）
- **「萬頁以上大型站點」** → **Hugo**（5 秒建置、成熟分類系統）
- **「GitHub Pages 零 CI 部署」** → **Jekyll**（原生支援）
- **「最小 JavaScript 與完全控制」** → **Eleventy**（預設零 JS）
- **「完全不想要 Node.js」** → **Hugo** 或 **Zola**（Go/Rust 單一二進位檔）
- **「Python 開發者」** → **Pelican**（Python/Jinja2）
- **「文件站需求」** → **Astro**（Starlight 主題）或 **MkDocs**（Material 主題）
- **「需要階層分類」** → **Hugo**（完整分類系統）或 **Hexo**（巢狀分類）

---

[^hugo-speed]: CloudCannon. (2025). The top five static site generators for 2025 and when to use them. Retrieved 2026-10-01, from https://cloudcannon.com/blog/the-top-five-static-site-generators-for-2025-and-when-to-use-them/
[^hugo-taxo]: The Hugo Authors. (n.d.). Taxonomies. Retrieved 2026-10-01, from https://gohugo.io/content-management/taxonomies/
[^zola-speed]: Talos Tools. (2026). Best Static Site Generators 2026. Retrieved 2026-10-01, from https://talos.tools/blog/best-static-site-generators-2026
[^zola-taxo]: Zola Authors. (n.d.). Taxonomies. Retrieved 2026-10-01, from https://www.getzola.org/documentation/content/taxonomies/
[^jekyll-github]: Jekyll Authors. (n.d.). Jekyll and GitHub Pages. Retrieved 2026-10-01, from https://jekyllrb.com/docs/github-pages/
[^jekyll-tag]: JekyllHub. (2026). Jekyll Tag and Category Pages Tutorial. Retrieved 2026-10-01, from https://jekyllhub.com/tutorial/2026/02/25/jekyll-tag-category-pages/
[^11ty-v3]: Eleventy Authors. (2024). Eleventy v3.0.0 Release. Retrieved 2026-10-01, from https://github.com/11ty/eleventy
[^11ty-collections]: Eleventy Authors. (n.d.). Collections. Retrieved 2026-10-01, from https://www.11ty.dev/docs/collections/
[^astro-2026]: TechRadar. (2026). Best static site generators. Retrieved 2026-10-01, from https://www.techradar.com/best/static-site-generators
[^astro-tag]: Astro Authors. (n.d.). Content Collections. Retrieved 2026-10-01, from https://docs.astro.build/en/guides/content-collections/
[^pelican-release]: Pelican Authors. (2026). Pelican v4.12.0 Release. Retrieved 2026-10-01, from https://github.com/getpelican/pelican
[^pelican-docs]: Pelican Authors. (n.d.). Content. Retrieved 2026-10-01, from https://docs.getpelican.com/en/stable/content.html
[^hexo-front]: Hexo Authors. (n.d.). Front-matter. Retrieved 2026-10-01, from https://hexo.io/docs/front-matter
[^mkdocs-tag]: MkDocs Material Authors. (n.d.). Setting up tags. Retrieved 2026-10-01, from https://squidfunk.github.io/mkdocs-material/setup/setting-up-tags/
[^nextjs-export]: Vercel. (n.d.). Static Exports. Retrieved 2026-10-01, from https://nextjs.org/docs/pages/building-your-application/deploying/static-exports
[^nextjs-blog]: Next.js Authors. (n.d.). Blog tutorial with Markdown. Retrieved 2026-10-01, from https://nextjs.org/learn-pages-router/basics/data-fetching
[^sveltekit-static]: Svelte Authors. (n.d.). adapter-static. Retrieved 2026-10-01, from https://kit.svelte.dev/docs/adapter-static