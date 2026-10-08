# VitePress 在大量文章（部落格）場景下的可行性分析

VitePress 是一款以文件（Documentation）為核心場景的靜態網站產生器（Static Site Generator, SSG），基於 Vite 與 Vue 建構。本報告探討將其用於部落格（Blog）——尤其是數百篇至數千篇文章等級規模——的可行性、已知限制與替代方案。

## 一、建置效能與規模瓶頸

### 1.1 實測數據

根據社群回報與官方維護者的說明，VitePress 的建置時間隨頁面數量成長呈現非線性上升：

| 規模 | 建置時間 | 資料來源 |
|------|----------|----------|
| ~25 頁 | 約 8–46 秒（依內容複雜度） | GitHub Discussion #1260[^1260] |
| ~100 頁 | 約 30–40 秒 | Hacker News 使用者回報[^hn] |
| 500 頁 | 19.4 秒（冷建置，4 核心 runner） | SSG Production 基準測試（VitePress 1.5）[^ssg] |
| ~26,000 頁（動態路由） | **OOM 記憶體耗盡崩潰**，即使配置 8 GB heap 也無法完成 | GitHub Issue #5134（2026-03）[^oom] |

官方維護者 brc-dd 在 Discussion #3189 中明確表示[^mnt]：

> 「如果你有數千個檔案，那麼現在你應該改用其他工具。」

### 1.2 瓶頸根源

- **Rollup 捆綁限制**：VitePress 底層使用 Rollup 進行靜態分析與打包，大量頁面會導致記憶體用量爆炸。社群期待未來 **rolldown**（Rust 實作的 bundler）能解決此問題，但尚無具體時間表[^mnt]。
- **Local Search 索引**：Issue #3377 紀錄了一個大型文件站點遭遇 **超過 4 小時的建置時間**，且最終產出 **空的搜尋索引**，原因是每個檔案被索引兩次，正規表達式標題解析在嵌入式 HTML 上失敗[^search]。
- **JavaScript Payload 大小**：VitePress 每個頁面約傳輸 **72 KB（壓縮後）** 的 Vue SPA 運行時，相比 Astro/Starlight 的約 18 KB 高出許多[^ssg]。

## 二、部落格所需功能支援度

VitePress 被設計為文件工具，官方已多次拒絕加入部落格專屬功能[^1838]。

| 功能 | VitePress 內建？ | 備註 |
|------|-----------------|------|
| **RSS/Atom feed** | ❌ 無 | 社群插件 `vitepress-plugin-rss` |
| **標籤/分類系統** | ❌ 無 | 需手動透過 data loader 實作 |
| **封存頁面** | ❌ 無 | 需手動建置 |
| **文章列表** | ❌ 無 | 無內建部落格索引佈局 |
| **留言系統** | ❌ 無 | 需整合 Disqus/utterances 等第三方 |
| **作者頁面** | ❌ 無 | 需手動實作 |
| **部落格分頁** | ❌ 無 | 需手動實作 |
| **部落格主題** | ❌ 無 | 數個社群主題可用（vitepress-blog, VPB theme） |
| **版本控制文件** | ❌ 無 | Docusaurus 內建此功能 |
| **自動側邊欄** | ❌ 無 | 需手動設定或使用 `vitepress-sidebar` 插件 |
| **本地搜尋** | ✅ 有 | 但在大規模站點表現不佳 |

### 社群解決方案

部分社群專案填補了缺口：

- [`vitepress-plugin-blog`](https://github.com/humanbydefinition/vitepress-plugin-blog) — 自動文章發現、部落格佈局、部落格元件
- [`vitepress-blogs-theme` (VPB)](https://chunge16.github.io/vitepress-blogs-theme/) — 部落格頁面、作者頁面、標籤頁面、封存導航
- [`vitepress-plugin-rss`](https://github.com/) — RSS feed 生成
- [`vitepress-sidebar`](https://github.com/) — 自動生成側邊欄

然而，引入這些插件仍無法解決底層的建置效能瓶頸。

## 三、與其他工具比較

| 面向 | VitePress | Hugo | Astro | Ghost | WordPress |
|------|-----------|------|-------|-------|-----------|
| 核心場景 | 文件 | 通用 SSG | 內容網站 | 部落格/CMS | 部落格/CMS |
| 100 頁建置速度 | 30–40 秒 | **< 1 秒** | 數秒 | N/A（動態） | N/A（動態） |
| 數千頁建置速度 | OOM／數小時 | **數秒** | 分鐘級 | N/A（動態） | N/A（動態） |
| 留言系統 | ❌ 需外掛 | ❌ 需外掛 | ❌ 需外掛 | ✅ 內建 | ✅ 內建 |
| RSS | ❌ 需插件 | ✅ 內建 | ✅ 內建 | ✅ 內建 | ✅ 內建 |
| 標籤/分類 | ❌ 需手動 | ✅ 內建 | ✅ 內建 | ✅ 內建 | ✅ 內建 |
| 管理後台 | ❌ 無 | ❌ 無 | ❌ 無 | ✅ 有 | ✅ 有 |
| 生態系 | 小（文件為主） | 大 | 中（成長中） | 中 | 極大 |
| JS 承載量 | ~72 KB | 極小 | ~18 KB | — | — |
| 託管方式 | 靜態 | 靜態 | 靜態 | 需伺服器 | 需伺服器 |

Hugo 在 SSG 建置速度上長期保持標竿地位，數百頁可在 1 秒內完成，且內建 RSS、分類系統與豐富主題生態[^css][^hn]。

## 四、結論與建議

### ✅ 適合使用 VitePress 的部落格場景

- 文章總數 **約 500 篇以下**
- 已身處 **Vue 生態系**，熟悉 Vue 客製化
- 願意 **自行建置部落格基礎設施**（標籤、RSS、封存）
- 希望獲得 **靜態站點的快速、安全、低成本部署**
- 內容以 **Markdown 檔案 + Git 版本控制** 管理

### ❌ 不適合使用 VitePress 的部落格場景

- 文章達 **數千篇以上**（面臨 OOM 崩潰或數小時建置）
- 需要 **管理後台** 撰寫文章（應考慮 WordPress 或 Ghost）
- 不希望 **手刻部落格基礎設施**（RSS、標籤、封存等）
- 團隊以 **React** 為主（應考慮 Docusaurus 或 Astro）
- 需要 **版本化文件 + 部落格** 並存（建議 Docusaurus）

### 推薦替代方案

- **Hugo**：大量文章部落格的首選，Go 語言實作，建置速度極快，內建 RSS 與分類
- **Astro**：內容為主的站點，多框架元件支援，JS 承載量極低
- **Ghost**：專職部落格平台，內建管理後台、留言、RSS、標籤
- **WordPress**：部落格標準，生態系最大，圖形化後台
- **Docusaurus**：React 生態中需要版本化文件 + 部落格的選擇

---

[^1260]: Vue.js. (n.d.). VitePress Discussion #1260: Build times for large sites. GitHub. Retrieved 2026-10-01, from https://github.com/vuejs/vitepress/discussions/1260
[^hn]: Hacker News. (2024). VitePress build time user report. Retrieved 2026-10-01, from https://news.ycombinator.com/item?id=39781090
[^ssg]: SSG Production. (2025). Choosing the right static site generator: VitePress vs Starlight benchmark (500 pages). Retrieved 2026-10-01, from https://www.ssg-production.com/choosing-the-right-static-site-generator-for-production/docs-frameworks-docusaurus-starlight-vitepress/vitepress-vs-starlight-for-vue-and-astro-teams/
[^oom]: Vue.js. (2026). GitHub Issue #5134: OOM crash with ~26,000 dynamic route pages. GitHub. Retrieved 2026-10-01, from https://github.com/vuejs/vitepress/issues/5134
[^mnt]: Vue.js. (n.d.). VitePress Discussion #3189: Can VitePress handle large projects? GitHub. Retrieved 2026-10-01, from https://github.com/vuejs/vitepress/discussions/3189
[^search]: Vue.js. (n.d.). GitHub Issue #3377: Local search indexing too slow for large sites. GitHub. Retrieved 2026-10-01, from https://github.com/vuejs/vitepress/issues/3377
[^1838]: Vue.js. (n.d.). GitHub Issue #1838: Blog layout feature request. GitHub. Retrieved 2026-10-01, from https://github.com/vuejs/vitepress/issues/1838
[^css]: CSS-Tricks. (n.d.). Comparing static site generator build times. Retrieved 2026-10-01, from https://css-tricks.com/comparing-static-site-generator-build-times/
[^docsio]: Docsio. (2025). VitePress in 2026 — Pros, Cons, Alternatives. Retrieved 2026-10-01, from https://docsio.co/blog/vitepress
[^eric]: Gardner, E. (2024). Blogging with VitePress. Retrieved 2026-10-01, from https://ericgardner.info/notes/blogging-with-vitepress-january-2024