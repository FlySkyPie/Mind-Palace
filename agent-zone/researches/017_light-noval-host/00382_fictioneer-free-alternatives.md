# Fictioneer 替代方案調查：非 WordPress、自由開源軟體

> Fictioneer（[Tetrakern/fictioneer](https://github.com/Tetrakern/fictioneer)）是一套基於 WordPress 的主題式網路小說（web fiction）發布平台，支援章節管理、閱讀器、TTS、OAuth 登入等特色功能。本報告針對**不依賴 WordPress** 的自由開源自架設替代方案進行調查，並以 GitHub Star 數量作為社群活躍度的參考指標。[^fictioneer-repo]

---

## 替代方案列表（依 Star 數排列）

### 1. Ghost — 55.5k ★

- **GitHub**: [TryGhost/Ghost](https://github.com/TryGhost/Ghost)
- **技術棧**: Node.js / MySQL / SQLite
- **授權**: MIT
- **定位**: 專業級出版平台。內建會員訂閱、電子報、SEO、主題系統。雖非專為 fiction 設計，但非常適合序列化小說發布 + 營利需求（付費會員、內容封鎖）。
- **部署**: 官方 CLI 或 Docker
- **相近 Fictioneer 功能**: OAuth 登入、內容封鎖、響應式佈局、SEO、客製化閱讀體驗（主題）
- **注意**: Ghost 更偏向部落格/出版而非專屬網路小說平台，缺少章節進度追蹤等 fiction 特化功能

### 2. WriteFreely — 5.3k ★

- **GitHub**: [writefreely/writefreely](https://github.com/writefreely/writefreely)
- **技術棧**: Go / SQLite / MySQL
- **授權**: AGPL
- **定位**: 輕量級 Markdown 寫作平台。支援 ActivityPub 聯邦（可與 Mastodon 互通）、多網誌帳號、OAuth 2.0。極低資源消耗（可在樹莓派運行），可視為最小可行 Fictioneer 替代。
- **部署**: 單一二進位檔，安裝簡易
- **相近 Fictioneer 功能**: OAuth、響應式、自架、SEO
- **注意**: 無章節/故事管理、無閱讀進度追蹤、無 TTS，功能較精簡

### 3. Plume — 2.2k ★

- **GitHub**: [Plume-org/Plume](https://github.com/Plume-org/Plume)（GitHub 已為鏡像，主要開發在 [git.joinplu.me](https://git.joinplu.me)）
- **技術棧**: Rust (Rocket) / PostgreSQL
- **授權**: AGPL
- **定位**: ActivityPub 聯邦式部落格引擎。單帳號可管理多個網誌，支援協作文章、多媒體管理。聯邦特性讓讀者可直接從 Mastodon 等平台跟隨。
- **相近 Fictioneer 功能**: 自架、多用戶、響應式
- **注意**: 維護不活躍（開發者時間有限）；無 fiction 特化功能

### 4. Publify — 1.9k ★

- **GitHub**: [publify/publify](https://github.com/publify/publify)
- **技術棧**: Ruby on Rails / MySQL / PostgreSQL / SQLite
- **授權**: MIT
- **定位**: 經典自架部落格引擎（前身 Typo，Rails 最老開源專案，自 2004 年）。多用戶、Markdown、社群功能。
- **相近 Fictioneer 功能**: 自架、多用戶、SEO、主題系統
- **注意**: 通用部落格平台，無 web fiction 特化功能

### 5. Booktype — 961 ★

- **GitHub**: [booktype/Booktype](https://github.com/booktype/Booktype)
- **技術棧**: Django (Python) / PostgreSQL
- **授權**: BSD
- **定位**: 協作式圖書編輯與出版平台。可匯入 DOCX/EPUB，輸出 print-ready PDF、EPUB、HTML。支援多人即時協作。
- **相近 Fictioneer 功能**: 自架、多用戶、EPUB 輸出
- **注意**: 偏向**圖書生產**（collaborative book production）而非**序列化在線發布**；不適合章節式連載

### 6. Tanu — 1 ★

- **GitHub**: [NewGaea/Tanu](https://github.com/NewGaea/Tanu)
- **技術棧**: Django 5.2 / Python 3.14 / PostgreSQL
- **授權**: AGPL
- **定位**: **唯一專為 web fiction 設計的非 WordPress 替代方案**。目標是自架式「獨立故事家園」，支援章節式圖書與圖書館管理。目前處於早期開發階段（專案重啟中）。
- **相近 Fictioneer 功能**: 章節管理、圖書館/收藏、EPUB 輸出、自架
- **注意**: 星數僅 1，屬於極早期專案，尚未有穩定版本

### 其他（不直接可比的寫作工具）

| 專案 | 類型 | 說明 |
|---|---|---|
| **OpenWrite** (55 ★) | AI 寫作平台 | React/Cloudflare Workers；AI 輔助小說寫作，需自備 API key |
| **Kindling** (70 ★) | 桌面寫作軟體 | Tauri/Svelte/Rust；offline-first 小說寫作工具，非發布平台 |
| **Web Novel Static Generator GUI** | 靜態網站產生器 | Python/Gradio；部署到 GitHub Pages，無伺服器依賴 |

---

## 綜合比較表

| 方案 | 類型 | 部署難度 | Star ★ | Fiction 特化 | 活躍維護 | 適合對象 |
|---|---|---|---|---|---|---|
| **Ghost** | 專業出版平台 | 中 (CLI/Docker) | 55.5k | ❌ | ✅ 非常活躍 | 想靠小說營利的作者 |
| **WriteFreely** | 輕量寫作平台 | 易 (單一二進位) | 5.3k | ❌ | ✅ 活躍 | 追求輕量簡約的作者 |
| **Plume** | 聯邦部落格引擎 | 中 (Rust/Docker) | 2.2k | ❌ | ⚠️ 低維護 | 需要聯邦功能的作者 |
| **Publify** | 傳統部落格引擎 | 中 (Rails) | 1.9k | ❌ | ⚠️ 低維護 | 熟悉 Rails 的作者 |
| **Booktype** | 協作圖書生產 | 中 (Django/Docker) | 961 | ❌ | ❌ 已不活躍 | 需要協作出版的團隊 |
| **Tanu** | **專屬 web fiction** | 中 (Django/Docker) | 1 | ✅ **唯一** | ✅ 早期但活躍開發 | 願意參與早期專案的作者 |

---

## 結論與建議

截至目前（2026-10），**不存在一個成熟、專為網路小說設計、且不依賴 WordPress 的自由開源替代方案**。

- **Tanu** 是唯一一個與 Fictioneer 目標完全一致的專案（獨立故事家園、章節管理、自架），但處於極早期，尚未具備 Fictioneer 的完整功能集。
- **Ghost** 在功能成熟度和社群規模上是遠超其他人的選擇，但需要自行適應其通用出版模式來發布 fiction。
- **WriteFreely** 是部署最簡易的選項，適合想要最小可行解決方案的作者。

若要取代 Fictioneer 的全部功能（章節管理、閱讀進度、TTS、OAuth、內容封鎖、EPUB 輸出），目前最務實的路徑是使用 **Ghost** 搭配客製主題與插件，或者考慮基於 **Tanu** 專案進行貢獻/自建。

---

## 參考資料

[^fictioneer-repo]: Tetrakern. (n.d.). *fictioneer* (Version 5.x) [WordPress theme for web fiction publishing]. GitHub. Retrieved 2026-10-01, from https://github.com/Tetrakern/fictioneer

[^ghost]: TryGhost. (n.d.). *Ghost* (Version 5.x) [Independent technology for modern publishing]. GitHub. Retrieved 2026-10-01, from https://github.com/TryGhost/Ghost

[^writefreely]: WriteFreely. (n.d.). *WriteFreely* (Version 0.x) [A clean, Markdown-based publishing platform made for writers]. GitHub. Retrieved 2026-10-01, from https://github.com/writefreely/writefreely

[^plume]: Plume-org. (n.d.). *Plume* [Federated blogging application, thanks to ActivityPub]. GitHub. Retrieved 2026-10-01, from https://github.com/Plume-org/Plume

[^publify]: Publify. (n.d.). *Publify* [A self hosted Web publishing platform on Rails]. GitHub. Retrieved 2026-10-01, from https://github.com/publify/publify

[^booktype]: Booktype. (n.d.). *Booktype* [Free, open source platform that produces beautiful, engaging books]. GitHub. Retrieved 2026-10-01, from https://github.com/booktype/Booktype

[^tanu]: NewGaea. (n.d.). *Tanu* [An open-source, self-hostable home for independent stories]. GitHub. Retrieved 2026-10-01, from https://github.com/NewGaea/Tanu

[^openwrite]: Ilrein. (n.d.). *OpenWrite* [Open-source AI-powered writing platform for novelists, screenwriters, and creative writers]. GitHub. Retrieved 2026-10-01, from https://github.com/ilrein/openwrite

[^kindling]: Smith-and-web. (n.d.). *Kindling* [Free, open-source fiction writing software]. GitHub. Retrieved 2026-10-01, from https://github.com/smith-and-web/kindling

[^writefreely-desc]: WriteFreely. (n.d.). *WriteFreely* [Official website]. Retrieved 2026-10-01, from https://writefreely.org

[^plume-desc]: Plume-org. (n.d.). *Plume* [Official website]. Retrieved 2026-10-01, from https://joinplu.me