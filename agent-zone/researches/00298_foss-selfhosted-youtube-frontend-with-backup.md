# FOSS 自架 YouTuber 前端 + 本地備份方案

## 研究目的

針對「具備自架（Self-hosted）能力」且「支援本地影片備份下載」的 FOSS（Free and Open Source）YouTuber 前端進行調查，以尋找同時具備觀看與封存能力的解決方案。

---

## 分類說明

這類工具可分為兩大類：

1. **純隱私前端（Privacy Frontend）**：代理 YouTube 內容、去廣告去追蹤，但不永久儲存影片。
2. **下載封存工具（Downloader / Archiver）**：將影片下載至本地磁碟，建立離線媒體庫。

部分專案同時具備兩者特性。

---

## 專案列表

### 1. TubeArchivist

- **定位**：自架 YouTube 媒體伺服器，最完整的一站式封存方案。
- **自架方式**：✅ Docker Compose（TubeArchivist + Elasticsearch + Redis），需約 2GB+ RAM。
- **本地備份下載**：✅ 永久下載至本地磁碟，保留完整中繼資料（標題、描述、縮圖、字幕、章節）。
- **關鍵功能**：基於 Elasticsearch 的全文字搜尋；頻道訂閱自動下載；多使用者看過進度追蹤；REST API；SponsorBlock 整合；以關鍵字過濾下載內容；Jellyfin/Plex 外掛；瀏覽器擴充套件（Companion）；Apprise 通知。
- **GitHub**：[https://github.com/tubearchivist/tubearchivist](https://github.com/tubearchivist/tubearchivist)[^ta]
- **Stars**：~8.5k
- **授權**：GPL-3.0

---

### 2. Pinchflat

- **定位**：輕量自含式 YouTube 媒體管理員，自動下載頻道/播放清單內容。
- **自架方式**：✅ 單一 Docker 容器，無外部依賴，非常輕量。
- **本地備份下載**：✅ 自動定期下載新內容；可依規則重新下載更高畫質。支援純音訊下載。
- **關鍵功能**：強大的命名/檔案組織系統；一級 Plex/Jellyfin/Kodi 支援；Podcast RSS feed；自動刪除舊內容；SponsorBlock 整合；自訂 yt-dlp 參數與生命週期腳本。無資料庫依賴。
- **GitHub**：[https://github.com/kieraneglin/pinchflat](https://github.com/kieraneglin/pinchflat)[^pinchflat]
- **Stars**：~5.4k
- **授權**：AGPL-3.0

---

### 3. MeTube

- **定位**：極簡 Web UI 包裝的 yt-dlp 下載工具。
- **自架方式**：✅ 單一 Docker 容器，部署非常簡單。
- **本地備份下載**：✅ 下載影片、音訊、字幕、縮圖至伺服器。支援單一影片、播放清單、完整頻道。
- **關鍵功能**：極簡 Web UI；支援 YouTube 與數百個其他網站（透過 yt-dlp）；訂閱功能（自動檢查新上傳）；自訂輸出資料夾/命名規則；瀏覽器擴充套件（Chrome/Firefox）/iOS Shortcut/Bookmarklet；硬體加速轉碼（Intel/AMD GPU）。
- **GitHub**：[https://github.com/alexta69/metube](https://github.com/alexta69/metube)[^metube]
- **Stars**：~14.9k
- **授權**：AGPL-3.0

---

### 4. TubeSync

- **定位**：YouTube 的 PVR（個人錄影機），類似 Sonarr 但專為 YouTube 設計。
- **自架方式**：✅ Docker/Podman 單一容器。Django 網頁應用。
- **本地備份下載**：✅ 定時排程下載至本地磁碟。失敗會以 back-off 機制重試。
- **關鍵功能**：類似 PVR 的排程索引與下載體驗；畫質/格式選擇；支援 Jellyfin/Plex 自動更新；可設定索引/下載頻率；SQLite 資料庫（易備份遷移）。
- **GitHub**：[https://github.com/meeb/tubesync](https://github.com/meeb/tubesync)[^tubesync]
- **Stars**：~2.8k
- **授權**：AGPL-3.0

---

### 5. ytdl-sub

- **定位**：命令列工具，自動下載 YouTube 內容並產生媒體中心適用的中繼資料。
- **自架方式**：✅ 純 CLI 工具，支援 Docker 部署。可選 code-server 作為 Web GUI。
- **本地備份下載**：✅ 下載並按頻道/播放清單組織檔案。產生完整 NFO 中繼資料、縮圖、海報。
- **關鍵功能**：YAML 格式訂閱設定；將 YouTube 頻道以電視劇形式組織（按日期/季）；beets API 音樂標籤整合；預設 Preset 快速上手；支援 SoundCloud、Bandcamp。
- **GitHub**：[https://github.com/jmbannon/ytdl-sub](https://github.com/jmbannon/ytdl-sub)[^ytdlsub]
- **Stars**：~3.0k
- **授權**：GPL-3.0

---

### 6. NewPipeWeb

- **定位**：基於 NewPipeExtractor 的全功能自架 YouTube/SoundCloud/PeerTube/Bandcamp/Odysee 前端，兼具觀看與下載能力。
- **自架方式**：✅ Docker Compose（Ktor 後端 + React 前端 + nginx）。另有 Tauri 桌面版應用。專案仍屬早期階段。
- **本地備份下載**：✅ 可將影片或音訊下載至伺服器，儲存於 `./downloads/` 目錄，並在 UI 中顯示下載進度。
- **關鍵功能**：支援多平台（YouTube、SoundCloud、PeerTube、Bandcamp、Odysee、media.ccc.de）；無帳號觀看；畫質選擇（144p–2160p）；純音訊模式；PiP；留言、字幕、訂閱；觀看進度記錄；SponsorBlock 整合；深色/淺色主題；資料匯出/匯入。
- **GitHub**：[https://github.com/chukjosh/NewPipeWeb](https://github.com/chukjosh/NewPipeWeb)[^npw]
- **Stars**：~8（早期專案）
- **授權**：GPL-3.0

---

### 7. Invidious（對照組 — 無備份）

- **定位**：最成熟的 YouTube 隱私前端。純代理，無本地儲存。
- **自架方式**：✅ Docker Compose。
- **本地備份下載**：❌ 無。串流來自 YouTube，不儲存影片。
- **關鍵功能**：去除廣告與追蹤；無需 Google 帳號訂閱頻道；RSS Feed；SponsorBlock；極簡 HTML/CSS UI。
- **GitHub**：[https://github.com/iv-org/invidious](https://github.com/iv-org/invidious)[^invidious]
- **授權**：AGPL-3.0

---

### 8. Piped（對照組 — 無備份）

- **定位**：較新的隱私前端，分離式架構（Kotlin 後端 + Vue.js 前端）。
- **自架方式**：✅ Docker Compose。
- **本地備份下載**：❌ 無。
- **GitHub**：[https://github.com/TeamPiped/Piped](https://github.com/TeamPiped/Piped)[^piped]
- **授權**：AGPL-3.0

---

### 9. yewtube（原 mps-youtube）

- **定位**：終端機 TUI 版 YouTube 播放與下載工具。
- **自架方式**：❌ 非伺服器，為本機 CLI 工具（透過 pip/pipx 安裝）。
- **本地備份下載**：✅ 可將影片/音訊下載至本地磁碟，支援格式與畫質選擇。
- **GitHub**：[https://github.com/mps-youtube/yewtube](https://github.com/mps-youtube/yewtube)[^yewtube]
- **授權**：GPL-3.0

---

## 綜合比較表

| 專案 | 分類 | 自架 | 本地備份 | Web UI | Docker | GitHub Stars |
|---|---|---|---|---|---|---|
| **TubeArchivist** | 媒體伺服器 | ✅ | ✅ | ✅ | ✅ | ~8.5k |
| **Pinchflat** | 媒體管理員 | ✅ | ✅ | ✅ | ✅ 單容器 | ~5.4k |
| **MeTube** | 下載器 | ✅ | ✅ | ✅ | ✅ 單容器 | ~14.9k |
| **TubeSync** | PVR 下載器 | ✅ | ✅ | ✅ | ✅ 單容器 | ~2.8k |
| **ytdl-sub** | CLI 下載+中繼資料 | ✅ | ✅ | 🔶 選配 | ✅ | ~3.0k |
| **NewPipeWeb** | 前端+下載器 | ✅ | ✅ | ✅ | ✅ | ~8 |
| **Invidious** | 純前端 | ✅ | ❌ | ✅ | ✅ | ~18.9k |
| **Piped** | 純前端 | ✅ | ❌ | ✅ | ✅ | ~9.9k |
| **yewtube** | CLI/TUI | 🔶 本機工具 | ✅ | ❌ TUI | ❌ | ~8.8k |

---

## 情境推薦

- **想完整封存頻道並可全文搜尋** → **TubeArchivist**（最成熟完整，但需較多資源）[^ta]
- **輕量自動下載、媒體中心整合** → **Pinchflat**（單一容器、無外部依賴）[^pinchflat]
- **只要一個簡單的 Web UI 下載器** → **MeTube**（極簡、星數最高、單一容器）[^metube]
- **像 Sonarr 一樣排程錄影** → **TubeSync**[^tubesync]
- **想將 YouTube 頻道以電視劇形式匯入 Plex/Jellyfin** → **ytdl-sub**[^ytdlsub]
- **想兼具觀看與下載的 Web 前端** → **NewPipeWeb**（但屬早期專案）[^npw]

---

## 參考資料

[^ta]: tube-archive. (n.d.). TubeArchivist — Your self-hosted YouTube media server. Retrieved 2026-09-27, from https://github.com/tubearchivist/tubearchivist
[^pinchflat]: Kierán E. (n.d.). Pinchflat — A lightweight self-contained YouTube media manager. Retrieved 2026-09-27, from https://github.com/kieraneglin/pinchflat
[^metube]: Alexta69. (n.d.). MeTube — Web GUI for yt-dlp. Retrieved 2026-09-27, from https://github.com/alexta69/metube
[^tubesync]: Meeb. (n.d.). TubeSync — PVR for YouTube. Retrieved 2026-09-27, from https://github.com/meeb/tubesync
[^ytdlsub]: Jmbannon. (n.d.). ytdl-sub — Automate downloading YouTube content with metadata. Retrieved 2026-09-27, from https://github.com/jmbannon/ytdl-sub
[^npw]: Chukjosh. (n.d.). NewPipeWeb — Self-hosted NewPipe experience in the browser. Retrieved 2026-09-27, from https://github.com/chukjosh/NewPipeWeb
[^invidious]: iv-org. (n.d.). Invidious — Invidious is an alternative front-end to YouTube. Retrieved 2026-09-27, from https://github.com/iv-org/invidious
[^piped]: TeamPiped. (n.d.). Piped — An alternative privacy-friendly YouTube frontend. Retrieved 2026-09-27, from https://github.com/TeamPiped/Piped
[^yewtube]: mps-youtube. (n.d.). yewtube — Terminal based YouTube player and downloader. Retrieved 2026-09-27, from https://github.com/mps-youtube/yewtube