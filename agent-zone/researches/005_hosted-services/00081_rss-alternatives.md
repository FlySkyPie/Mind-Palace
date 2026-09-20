# RSS 的現代替代方案：內容訂閱與聯合技術調查

## 摘要

RSS (Really Simple Syndication) 自 1999 年以來一直是網頁內容訂閱的事實標準，但隨著 XML 的技術老化、瀏覽器逐漸移除內建 RSS 支援，以及社群媒體的興起，多種現代化替代協定相繼出現。本報告調查八個主要替代方案——JSON Feed、WebSub、ActivityPub、Webmention、Micropub、AT Protocol (Bluesky)、Nostr 與 Atom——從技術架構、採用程度、使用案例與優缺點進行比較分析。

## 1. 背景

RSS 是一種基於 XML 的網頁 Feed 格式，讀者透過 Feed 閱讀器定期輪詢 (polling) 伺服器來檢查新內容[^rsswiki]。其核心問題包括：XML 冗長且不利於開發者解析、無即時更新機制、多版本碎片化（RSS 0.9/1.0/2.0 及 Atom），以及缺乏社交互動原語。Google Reader 於 2013 年關閉後，RSS 的主流使用顯著衰退[^rsshistory]。

然而 RSS 仍未被淘汰——Podcasting 依然運作於 RSS 之上，WordPress 等 CMS 平台持續輸出 RSS Feed。本報告調查的替代方案並非完全取代 RSS，而是在不同面向提供現代化改進。

## 2. 替代方案詳細分析

### 2.1 JSON Feed

| 面向 | 細節 |
|---|---|
| **類型** | Feed 格式 (JSON) |
| **發布時間** | 2017 年 5 月 (v1.0)；2020 年 8 月 (v1.1) |
| **作者** | Brent Simmons & Manton Reece |
| **MIME 類型** | `application/feed+json` |
| **規範** | jsonfeed.org |

JSON Feed 是 RSS/Atom 最直接的現代化替代，使用 JSON 而非 XML，保留 RSS 的核心概念但大幅降低開發者負擔[^jsonfeed]。規格自帶 `hubs` 欄位可搭配 WebSub 實現即時推送。

**採用情況**：NetNewsWire、NewsBlur、ReadKit 等閱讀器支援；NPR、Micro.blog 等出版商使用；Swift、Go、Rust、Python 等 30+ 語言皆有函式庫[^jsonfeedcode]。

**優點**：JSON 解析門檻低、規格清晰無歧義、支援多作者/多附件/頭像/分頁、容易擴展（`_` 前綴自訂欄位）、v1 向後相容 v1.1。

**缺點**：生態系遠小於 RSS、缺乏即時推送（可透過 WebSub 補足）、非主流平台採用率低。

### 2.2 WebSub（原名 PubSubHubbub）

| 面向 | 細節 |
|---|---|
| **類型** | 發佈-訂閱通知協定 (HTTP) |
| **發布時間** | 約 2009 年 (PubSubHubbub)；2017 年 10 月更名 WebSub |
| **狀態** | W3C 推薦標準 (2018 年 1 月) |
| **規範** | w3.org/TR/websub |

WebSub 解決 RSS 最嚴重的輪詢浪費問題——透過 Hub 實現即時推送。當發布者更新內容時通知 Hub，Hub 立即推送至所有訂閱者[^websub]。

**採用情況**：Blogger、WordPress.com、CNN、Medium、Feedly、Flipboard、NewsBlur 皆支援。Google 曾營運公共 Hub 於 pubsubhubbub.appspot.com。

**優點**：即時傳遞消除輪詢延遲與頻寬浪費、W3C 標準、可搭配任何 HTTP 內容類型（不限 RSS）、支援 HMAC 安全驗證。

**缺點**：需 Hub 基礎設施（集中式中介）、訂閱者需可公開存取的 Webhook 端點、非內容格式本身而是傳輸層。

### 2.3 ActivityPub

| 面向 | 細節 |
|---|---|
| **類型** | 去中心化社交網路協定 |
| **發布時間** | 2018 年 1 月 (W3C 推薦標準) |
| **基礎技術** | ActivityStreams 2.0 + JSON-LD |
| **網站** | activitypub.rocks |

ActivityPub 是 Fediverse（聯邦宇宙）的核心協定，定義了客戶端-伺服器 (C2S) 與伺服器-伺服器 (S2S) 兩組 API，實現跨站點內容推送與社交互動[^activitypub]。

**採用情況**：Mastodon (~1500 萬+ 使用者)、Pixelfed、PeerTube、Lemmy、BookWyrm 等 Fediverse 平台；Meta Threads (2024 年起部分支援)、Flipboard (2023)、Discourse (2025) 等大型企業加入；WordPress 有 ActivityPub 外掛[^activitypubadoption]。

**優點**：真正的推送式訂閱、雙向社交互動（按讚/回覆/轉發）、去中心化無單一控制點、W3C 標準、大型企業採用驗證。

**缺點**：實作複雜度高（多層規格疊加）、流行貼文可能導致意外 DDoS、跨站資料同步不一致、帳號遷移困難。

### 2.4 Webmention

| 面向 | 細節 |
|---|---|
| **類型** | 反向連結通知協定 (HTTP POST) |
| **發布時間** | 2017 年 1 月 (W3C 推薦標準) |
| **組織** | W3C / IndieWebCamp |
| **規範** | w3.org/TR/webmention |

Webmention 是一個極簡協定——當網站 A 連結到網站 B 時，A 發送 HTTP POST 至 B 的 Webmention 端點，內容僅包含來源與目標 URL。B 可選擇抓取 A 的頁面並顯示為留言或互動[^webmention]。

**優點**：極簡設計、W3C 標準、實現去中心化留言系統、取代舊的 XML-RPC Pingback。

**缺點**：非內容訂閱協定本身、需雙方支援、垃圾留言風險（需驗證機制）。

### 2.5 Micropub

| 面向 | 細節 |
|---|---|
| **類型** | 客戶端-伺服器內容建立 API (HTTP) |
| **發布時間** | 2017 年 5 月 (W3C 推薦標準) |
| **組織** | W3C / IndieWebCamp |
| **規範** | w3.org/TR/micropub |

Micropub 是 IndieWeb 生態中的發佈端協定——讓外部客戶端透過 API 在你的網站上建立、更新、刪除文章，無需使用網站管理介面[^micropub]。

**優點**：W3C 標準、簡單 RESTful API、支援多種內容類型（文章、書籤、活動、RSVP 等）、OAuth 2.0。

**缺點**：非訂閱/聯合協定、採用範圍限於 IndieWeb 社群。

### 2.6 AT Protocol (ATProto / Bluesky)

| 面向 | 細節 |
|---|---|
| **類型** | 去中心化社交網路協定 |
| **發布時間** | 2022 年 10 月公開 |
| **開發者** | Bluesky Social PBC |
| **網站** | atproto.com |

AT Protocol 採用模組化微服務架構：Personal Data Servers (PDS) 儲存使用者資料、Relays 爬取 PDS 聚合為 Firehose、AppViews 消費 Firehose 建構應用、Labelers 提供去中心化審查[^atproto]。

**採用情況**：Bluesky（數千萬使用者）；WordPress ATmosphere 外掛 (2026 年 5 月)；Flipboard 支援 Bluesky 登入；IETF 標準化進行中[^atprotoadoption]。

**優點**：真正的帳號可攜性（更換伺服器不失去身份/社交圖譜）、模組化架構、加密身份 (DIDs)、可自訂審查與演算法、IETF 標準化。

**缺點**：目前高度集中於 Bluesky 的基礎設施、did:plc 身份方法為單點故障、生態系仍不如 ActivityPub 成熟、所有資料目前公開。

### 2.7 Nostr

| 面向 | 細節 |
|---|---|
| **類型** | 去中心化通訊協定 (JSON Events) |
| **發布時間** | 2020 年 3 月 |
| **作者** | fiatjaf（化名） |
| **網站** | github.com/nostr-protocol/nostr |

Nostr 設計極簡：使用者以加密金鑰對簽署 JSON Event，廣播至 Relays（儲存與轉發事件的伺服器），客戶端從多個 Relays 查詢事件建構 Feed[^nostr]。

**採用情況**：約 1800 萬使用者；Jack Dorsey 捐贈約 1025 萬美元；Block (Square) 於 2026 年 7 月推出基於 Nostr 的 Buzz 平台。

**優點**：極簡協定設計、加密簽署無需信任伺服器、抗審查、內建 Bitcoin Lightning Network 小額支付 (Zaps)。

**缺點**：垃圾訊息嚴重、無內建身份系統（僅公鑰，UX 差）、生態系較小、與 Bitcoin/加密貨幣文化強烈關聯。

### 2.8 Atom（補充）

Atom（RFC 4287）是 IETF 提出的 XML Feed 格式（2005 年），設計上比 RSS 更嚴謹（明確的 MIME type `application/atom+xml`、完善的日期/語言處理），但在格式戰爭中輸給 RSS 2.0。至今多數 Feed 閱讀器仍支援 Atom[^atom]。

## 3. 綜合比較

| 協定 | 格式 | 模型 | 主要用途 | 標準狀態 | 採用程度 | 實作難度 |
|---|---|---|---|---|---|---|
| **RSS** | XML | 輪詢(Pull) | 部落格/新聞訂閱 | 無正式標準 | 🟢 普及 | 🟢 簡單 |
| **Atom** | XML | 輪詢(Pull) | Feed 格式 | IETF RFC 4287 | 🟡 中等 | 🟢 簡單 |
| **JSON Feed** | JSON | 輪詢(Pull) | Feed 格式 | 非正式 | 🟡 中等 | 🟢 簡單 |
| **WebSub** | HTTP | 推送(Push) | 即時通知傳輸 | ✅ W3C | 🟡 中等 | 🟡 中等 |
| **ActivityPub** | JSON-LD | 聯邦推送 | 去中心化社交 | ✅ W3C | 🟢 廣泛 | 🔴 複雜 |
| **Webmention** | HTTP POST | 連結通知 | 跨站互動 | ✅ W3C | 🟡 中等 | 🟢 簡單 |
| **Micropub** | JSON | API | 內容發佈 | ✅ W3C | 🟡 小眾 | 🟢 簡單 |
| **AT Protocol** | CBOR/JSON | Firehose | 去中心化社交 | IETF 進行中 | 🟡 成長中 | 🔴 複雜 |
| **Nostr** | JSON | Relay | 抗審查通訊 | 無 | 🟡 成長中 | 🟢 簡單 |

## 4. 結論與建議

RSS 並非已死，而是處於「休眠」狀態——它的簡單性仍是 Podcasting 與部落格聯合的骨幹。現代替代方案並非全面取代 RSS，而是在不同面向提供改進：

1. **只想現代化 Feed 格式**：採用 **JSON Feed**，與 RSS 並行輸出，滿足開發者偏好 JSON 的需求。

2. **需要即時推送**：為 RSS/JSON Feed 加上 **WebSub**，消除輪詢延遲。WordPress.com 與 Blogger 已內建支援。

3. **想參與去中心化社交網路**：採用 **ActivityPub**，這是目前最成熟且採用最廣泛的聯邦社交協定。Mastodon 的使用者可以直接「追蹤」你的部落格。

4. **追求最先進的架構與可攜性**：關注 **AT Protocol**，但需注意規格仍在 IETF 標準化過程中。

5. **需要抗審查發佈**：**Nostr** 提供最強的加密保證與最簡設計，但垃圾訊息問題尚待解決。

6. **最完整的個人網站方案**：IndieWeb 組合拳——**ActivityPub**（發佈與訂閱）+ **Webmention**（互動）+ **Micropub**（跨裝置發文）+ **Microsub**（閱讀）。

對多數出版者而言，務實的選擇是：**RSS + JSON Feed + WebSub**（最大相容性與即時傳遞），並可選 **ActivityPub** 外掛以觸及 Fediverse 讀者群。

---

[^rsswiki]: Wikipedia. (n.d.). RSS. Retrieved 2026-09-20, from https://en.wikipedia.org/wiki/RSS

[^rsshistory]: Google Reader shutdown and RSS decline. Referenced in multiple sources.

[^jsonfeed]: JSON Feed. (n.d.). Official site. Retrieved 2026-09-20, from https://www.jsonfeed.org/

[^jsonfeedcode]: JSON Feed. (n.d.). Code libraries. Retrieved 2026-09-20, from https://www.jsonfeed.org/code/

[^websub]: W3C. (2018-01-23). WebSub W3C Recommendation. Retrieved 2026-09-20, from https://www.w3.org/TR/websub/

[^activitypub]: W3C. (2018-01-23). ActivityPub W3C Recommendation. Retrieved 2026-09-20, from https://www.w3.org/TR/activitypub/

[^activitypubadoption]: Fediverse Observer. (n.d.). Fediverse network statistics. Retrieved 2026-09-20, from https://fediverse.observer/

[^webmention]: W3C. (2017-01-12). Webmention W3C Recommendation. Retrieved 2026-09-20, from https://www.w3.org/TR/webmention/

[^micropub]: W3C. (2017-05-10). Micropub W3C Recommendation. Retrieved 2026-09-20, from https://www.w3.org/TR/micropub/

[^atproto]: Bluesky. (n.d.). AT Protocol documentation. Retrieved 2026-09-20, from https://atproto.com/

[^atprotoadoption]: Kleppmann, M., Frazzetto, P., & others. (2024). Bluesky and the AT Protocol. arXiv:2402.03239. Retrieved 2026-09-20, from https://arxiv.org/html/2402.03239

[^nostr]: nostr-protocol. (n.d.). Nostr specification. Retrieved 2026-09-20, from https://github.com/nostr-protocol/nostr

[^atom]: IETF. (2005-12). The Atom Syndication Format (RFC 4287). Retrieved 2026-09-20, from https://datatracker.ietf.org/doc/html/rfc4287