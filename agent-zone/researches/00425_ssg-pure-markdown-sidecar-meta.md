# Markdown 純粹性を保つ SSG 解決方案研究

> 聚焦：不破壞 Markdown 語法、不嵌入特殊模板語法，使用額外元資料檔案的 FOSS 靜態網站生成器。

**撰寫日期**：2026-10-03

## 目錄

- [一、問題意識：為何要純粹的 Markdown？](#一問題意識為何要純粹的-markdown)
- [二、方案分類](#二方案分類)
- [三、原生支援側車檔（Sidecar）的 SSG](#三原生支援側車檔sidecar的-ssg)
  - [1. Nanoc — 正式的 `page.md` + `page.yaml` 側車模式](#1-nanoc--正式的-pagemd--pageyaml-側車模式)
  - [2. Nikola — 雙檔格式（Two-file format）](#2-nikola--雙檔格式two-file-format)
- [四、完全無 Frontmatter 的 SSG](#四完全無-frontmatter-的-ssg)
  - [3. Soupault — CSS 選擇器提取元資料](#3-soupault--css-選擇器提取元資料)
  - [4. doktri — 從檔案系統推斷元資料](#4-doktri--從檔案系統推斷元資料)
  - [5. medusa-ssg — 零 Frontmatter 設計](#5-medusa-ssg--零-frontmatter-設計)
- [五、元資料集中管理型 SSG](#五元資料集中管理型-ssg)
  - [6. MkDocs — 集中式 YAML 配置](#6-mkdocs--集中式-yaml-配置)
  - [7. mdBook — 最小元資料干擾](#7-mdbook--最小元資料干擾)
- [六、部分支援側車檔案的 SSG](#六部分支援側車檔案的-ssg)
  - [8. Lektor — 附件側車模式](#8-lektor--附件側車模式)
- [七、對照總表](#七對照總表)
- [八、選擇建議](#八選擇建議)
- [參考資料](#參考資料)

---

## 一、問題意識：為何要純粹的 Markdown？

許多主流 SSG（Hugo、Jekyll、Zola、11ty 等）要求在 Markdown 檔案開頭嵌入 YAML/TOML Frontmatter：

```markdown
---
title: 我的文章
date: 2026-10-03
tags: [技術, SSG]
---

# 實際內容從這裡開始...
```

這種做法有以下問題：

1. **破壞可攜性**：該 Markdown 檔案無法直接在其他不認識 Frontmatter 的工具中正確顯示（如 GitHub 預覽、Pandoc、其他 SSG）。
2. **污染內容**：元資料與內容混合，非內容創作者關心的資訊強行進入檔案頭部。
3. **語法高亮干擾**：部分編輯器對 `---` 分隔的 YAML Frontmatter 處理不一致。

本報告尋找的解決方案——使用**側車檔（sidecar file）**—將 Markdown 內容與元資料分離，讓 `.md` 檔案保持純粹、可攜、標準。[^wiki-ssg]

---

## 二、方案分類

| 類別 | 說明 | 代表方案 |
|------|------|----------|
| **原生側車檔** | 正式支援在 Markdown 旁放置獨立的 YAML/META 檔案 | Nanoc、Nikola |
| **完全無 Frontmatter** | 根本不使用 Frontmatter，改從 HTML/CSS、檔案路徑、檔名推斷元資料 | Soupault、doktri、medusa-ssg |
| **集中管理** | 頁面元資料在站點層級的 YAML/Toml 配置檔案中定義，Markdown 不需任何元資料 | MkDocs、mdBook |
| **部份支援** | 以自有格式為主，但附件或部分情境可透過側車模式補充元資料 | Lektor |

---

## 三、原生支援側車檔（Sidecar）的 SSG

### 1. Nanoc — 正式的 `page.md` + `page.yaml` 側車模式

- **語言**：Ruby
- **授權**：MIT

Nanoc 是一套 Ruby 生態的 SSG，**正式支援且文檔明確記錄** 了側車元資料檔案模式。使用者可以將元資料存放在與 Markdown 同名的 `.yaml` 檔案中。

> 「如果你不喜歡在每頁頂端放一段元資料（可能是因為它破壞了語法高亮），你可以將元資料放在一個與頁面同名的 YAML 檔案中。例如，`content/about.html` 頁面的元資料可以存放在 `content/about.yaml`。」[^nanoc-tutorial]

> 「元資料也可以存放在一個獨立的檔案（即『meta file』）中，使用相同的基本檔名但副檔名為 `.yaml`。這對二進位項目是必要的。」[^nanoc-data-sources]

目錄結構範例：

```
content/
  about.md          # 純粹的 Markdown，無任何元資料
  about.yaml        # 側車元資料
  blog/
    hello-world.md
    hello-world.yaml
```

Nanoc 也支援行內 Frontmatter（YAML 或 TOML），但側車 `.yaml` 模式是選擇性的、明確的、且文檔完善的做法。這使其成為「保持 Markdown 純粹」的最佳首選之一。

---

### 2. Nikola — 雙檔格式（Two-file format）

- **語言**：Python 3
- **授權**：MIT

Nikola 是一套「batteries included」哲學的 Python SSG。它提供一種 **雙檔格式**，透過 `-2` 旗標啟用，建立兩個檔案：

- `posts/my-post.md` — 純粹 Markdown 內容
- `posts/my-post.meta` — 元資料檔案（reST-style，但也支援 YAML/TOML）

> 「預設情況下，該檔案（Markdown）也會包含一些關於文章的額外資訊（元資料）。透過使用 `-2` 選項，可以將元資料放在一個單獨的檔案中。」[^nikola-handbook]

`.meta` 檔案範例：

```
.. title: 如何賺錢
.. slug: how-to-make-money
.. date: 2012-09-15 19:52:05 UTC
.. tags: 財務, 投資
```

而對應的 Markdown 檔案中就是完全純粹的內容，沒有任何元資料污染。

Nikola 也支援行內模式，但 `-2` 雙檔格式是官方支援的做法。Nikola 生態中也存在用於批量升級/轉換元資料格式的插件。[^nikola-plugin]

---

## 四、完全無 Frontmatter 的 SSG

### 3. Soupault — CSS 選擇器提取元資料

- **語言**：OCaml（單一靜態二進位檔）
- **授權**：MIT

Soupault 是最激進且最符合「純粹 Markdown」哲學的 SSG。它**完全不使用 Frontmatter**，也沒有內建的內容模型。

Soupault 的做法是：先將 Markdown（或任何輸入格式）轉換為 HTML，然後**透過 CSS 選擇器從 HTML 元素樹中提取元資料**。

> 「與大多數靜態網站生成器不同，Soupault 不使用 Frontmatter，也沒有內建的內容模型，而是允許你直接從 HTML 本身提取元資料，並將內容模型欄位映射到 CSS 選擇器。」[^soupault]

設定範例（`soupault.toml`）：

```toml
[index.fields.title]
  selector = ["h1#post-title", "h1"]

[index.fields.date]
  selector = ["time#post-date", "time"]
  extract_attribute = "datetime"
  fallback_to_content = true

[index.fields.tags]
  selector = "meta[name='keywords']"
  extract_attribute = "content"
```

這表示你的 Markdown 檔案中完全不需要任何元資料標記——只需撰寫標準的 Markdown，Soupault 會從產生的 HTML 中抓取 `<h1>` 作為標題、從 `<time>` 的 `datetime` 屬性提取日期等。

因為 Soupault 是在 HTML 元素樹層級操作，所以這種方式**與輸入格式無關**——Markdown、reStructuredText、AsciiDoc、手寫 HTML 都同樣有效。[^soupault-ref]

---

### 4. doktri — 從檔案系統推斷元資料

- **語言**：Go
- **授權**：MIT

doktri 是一個以 Go 撰寫的輕量 SSG，其設計哲學明確拒絕 Frontmatter：

> 「這裡沒有 Frontmatter。我喜歡擁有 *純粹* 的 Markdown 內容，使其可以在任何地方使用而無需修改。Frontmatter 讓這變得困難。」[^doktri]

doktri 的做法：

- **日期**：從檔名前綴取得（如 `2024-01-15-hello-world.md`）
- **架構**：目錄樹被視為節點樹，可在模板中遍歷
- **站點元資料**：在專案根目錄放一個 `meta.yaml` 處理全域設定
- **每個頁面的特定元資料**：無——頁面標題取自第一個 `# 標題`

這是一種極簡哲學：無 Frontmatter、無側車檔，完全依賴慣例。

---

### 5. medusa-ssg — 零 Frontmatter 設計

- **語言**：Python
- **授權**：MIT

medusa-ssg 是一個 Python SSG，核心功能是「零 Frontmatter 需求」：

> 「零 Frontmatter 需求——標題、日期、標籤和描述皆從檔案名稱和內容推導。」[^medusa]

其做法：

- **標題**：取自檔案中的第一個 `#` 標題
- **日期**：從檔名日期前綴推斷（如 `2024-01-15-hello-world.md`）
- **草稿**：透過 `_` 前綴標記檔案或資料夾
- **選擇性 Frontmatter**：需要自訂元資料時，可選用 YAML Frontmatter
- **額外資料**：全域 YAML 資料檔案放在 `data/` 目錄

medusa-ssg 的策略是：預設不需要任何元資料，純粹從內容與檔案系統取得所需資訊；僅在需要自訂元資料時才選擇性使用 Frontmatter。

---

## 五、元資料集中管理型 SSG

### 6. MkDocs — 集中式 YAML 配置

- **語言**：Python
- **授權**：BSD 2-Clause

MkDocs 是文件網站專用的 SSG，其設計使得**所有 Markdown 檔案都可以完全沒有 Frontmatter**。導航結構和站點元資料統一在 `mkdocs.yml` 中定義：

```yaml
site_name: 我的文件
nav:
  - 首頁: index.md
  - 關於: about.md
  - 部落格:
    - 第一篇文章: blog/first.md
```

Markdown 檔案因此可以保持純粹：

```markdown
# 我的第一篇文章

這是純粹的 Markdown。
```

頁面標題取自第一個 `#` 標題。MkDocs 也支援選擇性的 YAML Frontmatter（透過 `---`），但完全不用也不影響任何功能。[^mkdocs]

這對文件網站非常適合，但對於需要豐富每頁元資料（如多標籤、自訂佈局、摘要等）的一般網站可能不夠靈活。

---

### 7. mdBook — 最小元資料干擾

- **語言**：Rust
- **授权**：MPL 2.0

mdBook 是 Rust 社群廣泛使用的文件工具（如 Rust 官方 Book 系列）。其結構：

- **`book.toml`**：全域配置
- **`SUMMARY.md`**：定義文件目錄和導航
- **個別 `.md` 檔案**：純粹的 Markdown，完全不需要元資料

mdBook 從內容取得頁面標題（第一個 `#` 標題），無需任何 Frontmatter。它的用途限制在文件/書籍場景，而非通用網站。[^mdbook]

---

## 六、部分支援側車檔案的 SSG

### 8. Lektor — 附件側車模式

- **語言**：Python（管理後臺含 Node.js）
- **授權**：BSD 3-Clause

Lektor 以自有 `.lr` 格式聞名（使用 `---` 分隔元資料欄位），但對於「附件」支援一種側車模式。

從 Lektor 附件文檔：

> 「附件也可以擁有元資料。你需要建立一個與附件同名的 Lektor 內容檔案，附加額外的 `.lr` 副檔名。」[^lektor]

例如，`image.jpg` 可以搭配 `image.jpg.lr`：

```
_model: image
---
description: 美麗的日落
---
photographer: 王先生
```

此外，存在第三方插件 `lektor-markdown-from-file`，允許將 Markdown 內容放在獨立的 `_contents.md` 檔案中，讓 `contents.lr` 僅包含元資料。[^lektor-plugin]

Lektor 並非為「純粹 Markdown」而設計，但它顯示出在特定情境下（如附件）側車模式的存在。

---

## 七、對照總表

| SSG | 元資料處理方式 | Markdown 純粹度 | 語言 | 授權 |
|-----|---------------|----------------|------|------|
| **Soupault** | CSS 選擇器從 HTML 提取 | ✅ 100% 純粹 | OCaml | MIT |
| **Nanoc** | `page.md` + `page.yaml` 側車 | ✅ 100% 純粹 | Ruby | MIT |
| **Nikola** | `page.md` + `page.meta` 雙檔 | ✅ 100% 純粹 | Python | MIT |
| **doktri** | 檔名/目錄結構推斷 | ✅ 100% 純粹 | Go | MIT |
| **medusa-ssg** | 檔名/內容推斷，可選 Frontmatter | ✅ 100% 純粹（預設） | Python | MIT |
| **MkDocs** | 集中式 `mkdocs.yml` | ✅ 100% 純粹（文件適用） | Python | BSD 2C |
| **mdBook** | `book.toml` + `SUMMARY.md` | ✅ 100% 純粹（書籍適用） | Rust | MPL 2.0 |
| **Lektor** | 自有 `.lr` 格式；附件側車 | ◐ 部分支援 | Python | BSD 3C |

---

## 八、選擇建議

- **最純粹的哲學** → **Soupault**。完全不使用 Frontmatter，也不強迫任何慣例。你的 Markdown 可以完全保持原樣，SSG 自行從 HTML 找出元資料。
- **最明確的側車檔支援** → **Nanoc**。文檔中正式記錄 `page.md` + `page.yaml` 的做法，是「保留純粹 Markdown + 側車 YAML」模式的首選。
- **Python 使用者的側車模式** → **Nikola**。`nikola new_post -2` 直接建立雙檔，生態系成熟。
- **文件網站、只需極簡配置** → **MkDocs**。完全不需 Frontmatter，所有導航資訊集中在一個 YAML 檔案。
- **極簡主義者** → **doktri** 或 **medusa-ssg**。依賴檔案系統慣例，無任何元資料機制。
- **書籍/文件** → **mdBook**。Rust 生態，零元資料污染。

---

## 參考資料

[^wiki-ssg]: Wikipedia. (n.d.). Static site generator. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Static_site_generator

[^nanoc-tutorial]: Nanoc. (n.d.). Nanoc tutorial. Retrieved 2026-10-03, from https://nanoc.denisdefreyne.com/doc/tutorial/

[^nanoc-data-sources]: Nanoc. (n.d.). Data sources. Retrieved 2026-10-03, from https://nanoc.denisdefreyne.com/doc/data-sources/

[^nikola-handbook]: Nikola. (n.d.). Nikola handbook — Two-file format. Retrieved 2026-10-03, from https://getnikola.com/handbook.html

[^nikola-plugin]: wirew0rm. (n.d.). nikola-plugins — upgrade_metadata. Retrieved 2026-10-03, from https://github.com/wirew0rm/nikola-plugins/tree/master/v7/upgrade_metadata

[^soupault]: Soupault. (n.d.). Soupault — Static site generator. Retrieved 2026-10-03, from https://soupault.app/

[^soupault-ref]: Soupault. (n.d.). Soupault reference manual — Metadata extraction and rendering. Retrieved 2026-10-03, from https://soupault.app/reference-manual/

[^doktri]: bluebrown. (n.d.). doktri. Retrieved 2026-10-03, from https://github.com/bluebrown/doktri

[^medusa]: wusher. (n.d.). medusa-ssg. Retrieved 2026-10-03, from https://github.com/wusher/medusa-ssg

[^mkdocs]: MkDocs. (n.d.). MkDocs — Project documentation with Markdown. Retrieved 2026-10-03, from https://www.mkdocs.org/

[^mdbook]: mdBook. (n.d.). mdBook — A utility to create books from Markdown files. Retrieved 2026-10-03, from https://github.com/rust-lang/mdbook

[^lektor]: Lektor. (n.d.). Lektor — Attachments. Retrieved 2026-10-03, from https://www.getlektor.com/docs/content/attachments/

[^lektor-plugin]: tealok-tech. (n.d.). lektor-markdown-from-file. Retrieved 2026-10-03, from https://github.com/tealok-tech/lektor-markdown-from-file