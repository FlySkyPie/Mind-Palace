# VitePress 是否支援標籤 (Tags/Labels) 功能？

## 結論

**VitePress 本身沒有內建的標籤或分類系統**。VitePress 本質上是設計為文件網站產生器，並非部落格平台。但社群有多種成熟方案可實現標籤功能。

---

## 1. 可用的基礎設施

雖然內建無標籤功能，VitePress 提供幾個可藉此打造標籤系統的底層機制[^vp-frontmatter][^vp-routing][^vp-data-loading]：

| 機制 | 說明 |
|------|------|
| **Frontmatter** | 可在 Markdown 檔案中放入任意 YAML 欄位（如 `tags: [vue, tutorial]`），透過 `useData()` 的 `frontmatter` 屬性存取 |
| **customData** | 在設定檔中注入自訂資料，可在主題元件中以 `$site.customData` 取得 |
| **動態路由 (Dynamic Routes)** | 透過 `[param].paths.js` 系統在建置時從資料產生頁面 |
| **Data Loaders** | 建置時掃描檔案並彙整 frontmatter 資料的工具 |

---

## 2. 現有套件與主題

### 2.1 vitepress-tags[^vitepress-tags]

最輕量的套件，掃描 Markdown 檔案的 frontmatter，建立標籤集合注入 `customData`，可在導航列或側邊欄使用。

```yaml
---
tags: collab
---
```

```js
// 在 .vitepress/config.ts 中
import tags from 'vitepress-tags'
export default {
  customData: {
    pages: tags()
  }
}
```

### 2.2 vitepress-plugin-blog[^vitepress-plugin-blog]

完整部落格外掛，內建標籤支援。提供 `useBlogPosts()` composable 暴露 `allTags` 和 `BlogFilters` 元件進行標籤篩選。

```yaml
---
blogPost: true
title: 我的第一篇部落格
tags:
  - vitepress
  - 教學
---
```

### 2.3 vitepress-theme-ououe[^vitepress-theme-ououe]

部落格主題，支援標籤和分類。需手動建立 `tag.md` 和 `category.md` 搭配特定佈局。

```yaml
---
tags:
  - vitepress
categories:
  - blog
---
```

### 2.4 @lando/vitepress-theme-default-plus[^lando-theme]

擴充預設主題，內建標籤系統。支援 URL 參數篩選（`/all.html?tag=obscure`）和自訂標籤樣式。

```yaml
---
tags:
  - obscure
  - 測試標籤
---
```

### 2.5 vitepress-blogs-theme[^vitepress-blogs-theme]

完整部落格主題，支援作者、標籤、分類和歸檔功能。

---

## 3. 自幹方案

### 方案 A：Frontmatter + 自訂 Vue 元件

最簡單：在 frontmatter 寫 tags，在自訂佈局元件中讀取並顯示。

### 方案 B：Data Loaders 彙整標籤[^vp-data-loading]

建立 `.data.ts` 掃描所有 Markdown 檔案的 frontmatter，建立標籤索引，然後在任何頁面或元件中匯入。

### 方案 C：動態路由產生標籤頁面[^vp-routing]

建立 `tags/[tag].md` 搭配 `tags/[tag].paths.js`，在建置時掃描所有檔案的標籤並自動產生對應頁面。

### 方案 D：customData + 建置腳本

編寫腳本掃描 Markdown 檔案、彙整標籤資料，注入 VitePress 設定的 `customData`。

---

## 4. 各方案比較

| 方案 | 複雜度 | 標籤頁面 | 標籤篩選 | 自訂程度 | 適合 |
|------|--------|----------|----------|----------|------|
| Frontmatter 顯示 | 低 | ❌ | ❌ | 高 | 純顯示 |
| Data Loaders | 中 | 自建 | 自建 | 高 | 自訂需求 |
| 動態路由 | 中高 | ✅ | 有限 | 高 | 靜態標籤頁 |
| vitepress-tags | 低 | ❌(僅導航) | 需自建 | 中 | 導航集合 |
| vitepress-plugin-blog | 低 | ✅內建 | ✅內建 | 高 | 完整部落格 |
| vitepress-theme-ououe | 中 | ✅ | ✅ | 受限 | 標籤部落格 |
| @lando/theme-default-plus | 中 | ✅URL篩選 | ✅ | 高 | 文件+集合 |

---

## 參考資料

[^vp-frontmatter]: VitePress Team. (n.d.). Frontmatter — VitePress. Retrieved 2026-10-03, from https://vitepress.dev/guide/frontmatter

[^vp-routing]: VitePress Team. (n.d.). Routing: Dynamic Routes — VitePress. Retrieved 2026-10-03, from https://vitepress.dev/guide/routing#dynamic-routes

[^vp-data-loading]: VitePress Team. (n.d.). Data Loading — VitePress. Retrieved 2026-10-03, from https://vitepress.dev/guide/data-loading

[^vitepress-tags]: davay42. (n.d.). vitepress-tags. Retrieved 2026-10-03, from https://github.com/davay42/vitepress-tags

[^vitepress-plugin-blog]: humanbydefinition. (n.d.). vitepress-plugin-blog. Retrieved 2026-10-03, from https://github.com/humanbydefinition/vitepress-plugin-blog

[^vitepress-theme-ououe]: tolking. (n.d.). vitepress-theme-ououe. Retrieved 2026-10-03, from https://tolking.github.io/vitepress-theme-ououe/posts/tag

[^lando-theme]: Lando. (n.d.). Tagging Shit — @lando/vitepress-theme-default-plus. Retrieved 2026-10-03, from https://vitepress-theme-default-plus.lando.dev/guides/tagging-shit.html

[^vitepress-blogs-theme]: chunge16. (n.d.). vitepress-blogs-theme. Retrieved 2026-10-03, from https://github.com/chunge16/vitepress-blogs-theme