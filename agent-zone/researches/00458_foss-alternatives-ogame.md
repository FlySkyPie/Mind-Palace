# OGame 自由開源替代方案調查報告

## 摘要

本報告調查 OGame（一款網頁版太空主題多人即時策略遊戲）之自由開源替代方案。經網路搜尋與專案分析，共找出 **9 個** 相關專案，其中 **OGameX** 是最活躍且功能最完整的直接克隆，**OGame Open Source** 保留了經典 v0.84 原版體驗，**OGame Vue TS** 則提供單機離線遊玩選項。

## 背景

OGame 於 2005 年上線，是一款經典的瀏覽器-based 太空 MMORTS，玩家管理星球、建造艦隊、探索銀河系並與其他玩家戰鬥。由於官方版本歷經多次更新（包含 2022 年的 Lifeforms 更新）且引入付費機制，部分社群轉而開發開源替代方案，以保留經典玩法或提供自託管選項。

## 研究結果

### 1. OGameX — 最現代且活躍的直接克隆

| 屬性 | 內容 |
|------|------|
| **描述** | 從頭使用 **Laravel 13 (PHP 8.5)** 與 **MariaDB** 重建的 OGame 開源克隆。目標重現 2022 年 Lifeforms 更新前的 OGame。戰鬥引擎使用 **Rust**（透過 PHP FFI），比純 PHP 快約 200 倍。 |
| **GitHub** | <https://github.com/lanedirt/OGameX> |
| **星數** | ⭐ 321 |
| **授權** | MIT |
| **Live Demo** | <https://main.ogamex.dev> (nightly) • <https://release.ogamex.dev> (stable v0.14.0) |
| **狀態** | ✅ **非常活躍** — 2,815+ 提交，頻繁發布版本，活躍 Discord 社群 |
| **語言** | PHP (Laravel) + Rust (戰鬥引擎) |

**主要功能：**
- 行星管理、建築、研究、造船廠、防禦、銀河系視圖、排行榜、訊息系統
- 所有艦隊任務：運輸、部署、殖民、間諜、攻擊、回收、ACS、毀月
- Rust-based 高效能戰鬥引擎（亦有 PHP 備用方案）
- 月亮建立（從殘骸場）、月球建築、月掃描器、跳躍門
- 殘骸場 / 太空船塢
- 職業系統（如 OGame）
- 遊戲內聊天、好友、筆記、艦隊模板
- 遠征任務（多種結果）
- 聯盟與 ACS 艦隊派遣
- 多語言支援
- 管理員面板
- **無付費取勝** — 所有暗物質功能均為非商業用途
- Docker-based 安裝
- 有明確的功能開發路線圖 [^ogamex]

---

### 2. OGame Open Source — 經典 v0.84 復刻

| 屬性 | 內容 |
|------|------|
| **描述** | 忠實重現 **OGame v0.84**，保留 2000 年代原始風格與操作介面。所有遊戲機制、資源成本、生產時間與行星分佈演算法皆與原版一致。 |
| **GitHub** | <https://github.com/ogamespec/ogame-opensource> |
| **星數** | ⭐ 132 |
| **授權** | CC0-1.0 |
| **狀態** | ✅ **活躍** — 1,839+ 提交，正在進行大規模重構（PHPStan、模組引擎） |
| **語言** | PHP（C-based 戰鬥引擎，附 PHP 備用方案） |

**主要功能：**
- **原始遊戲機制** — 100% 忠於經典 OGame 0.84
- 使用 **C 語言** 編寫的快速戰鬥引擎（附 PHP 備用方案）
- 原始遠征系統（三倍暗物質機率）
- ACS 支援
- 整合銀河系工具
- CRON-less 事件佇列（可選 CRON 模式）
- 修正若干原始遊戲錯誤（錯誤的第 10 行星、ACS 召回延遲等）
- **所有高級功能免費**（暗物質、軍官、貿易商）
- 多語言：俄語、英語、德語
- 2026 年仍有活躍開發 [^ogame-opensource]

---

### 3. OGame Vue TS — 現代單機瀏覽器版

| 屬性 | 內容 |
|------|------|
| **描述** | 使用 **Vue 3**、**TypeScript** 與 **Vite** 建立的 **單人** OGame 致敬版。完全在瀏覽器中執行，資料儲存於本地（AES 加密）。支援 Android（透過 Capacitor）與桌面（透過 Electron）。 |
| **GitHub** | <https://github.com/setube/ogame-vue-ts> |
| **星數** | ⭐ 516 |
| **授權** | CC-BY-NC 4.0 |
| **Live Demo** | <https://ogame-vue-ts.wenzi.games/> |
| **狀態** | ✅ **非常活躍** — 220+ 提交 |
| **語言** | TypeScript / Vue 3 / Go（後端服務） |

**主要功能：**
- **單人遊玩** — 無 MMO/多人模式，所有資料存於本地
- 建築管理、科技研究、艦隊管理、防禦系統
- 軍官系統與戰鬥模擬器
- 銀河系視圖探索
- 從殘骸場生成月亮
- 佇列管理（建築/研究）
- 深色/淺色主題
- AES 加密本地儲存
- Android 與 Electron 桌面支援
- ⚠️ **非多人遊戲**— 僅供離線練習或學習遊戲機制 [^ogame-vue-ts]

---

### 4. XG Proyect v4 — Laravel OGame 框架

| 屬性 | 內容 |
|------|------|
| **描述** | 使用 Laravel 重建的 OGame 克隆框架。v4 版為完全重寫版本，v3.x（CodeIgniter）已封存。 |
| **GitHub (v4)** | <https://github.com/XGProyect/XGProyect> |
| **星數 (v4)** | ⭐ 14 |
| **授權** | GPL-3.0 |
| **Live Server** | <https://www.xgproyect.net/> |
| **狀態** | ⚠️ **中等** — v4 較新但星數較少；v3 已封存 |
| **語言** | PHP (Laravel) |

**主要功能：**
- 完整 OGame 克隆框架
- PHP 8.2+、MySQL/MariaDB
- Docker 支援（Laravel Sail）
- Mailpit 郵件測試
- 社群活躍度低於 OGameX [^xgproyect]

---

### 5. 2Moons — OG 開源 OGame 框架

| 屬性 | 內容 |
|------|------|
| **描述** | 最早期的開源瀏覽器遊戲框架，源自 XNova → UGamela 血脈，為大多數 OGame 克隆的始祖。 |
| **GitHub** | <https://github.com/jkroepke/2Moons> |
| **星數** | ⭐ 160 |
| **授權** | MIT |
| **狀態** | ❌ **已封存**（2021 年 11 月）— 不再開發 |
| **語言** | PHP |

**主要功能：**
- 完整 OGame 克隆引擎
- 為 SteemNova、New-Star、pr0game 等專案的基礎
- ⚠️ 不建議新專案使用 — 建議使用其活躍分支（SteemNova、New-Star） [^2moons]

---

### 6. SteemNova — 區塊鏈整合 OGame 克隆

| 屬性 | 內容 |
|------|------|
| **描述** | 基於 2Moons 的多人瀏覽器遊戲，整合 **Steem 區塊鏈** 進行身份驗證與獎勵。 |
| **GitHub** | <https://github.com/steemnova/steemnova> |
| **星數** | ⭐ 107 |
| **授權** | AGPL-3.0 |
| **Live Server** | <https://steemnova.intinte.org> |
| **狀態** | ⚠️ **低活躍度** — 最近提交僅為維護更新 |
| **語言** | PHP |

**主要功能：**
- OGame 克隆搭配 **Steem 區塊鏈登入**（SteemConnect）
- 頂尖聯盟獎勵（透過 SBD）
- 完整遊戲引擎：建築、艦隊、戰鬥、研究
- 多語言支援（DE、EN、ES、FR、PL、PT、RU、TR）
- 遊戲內 AJAX 聊天
- 2Moons 的活躍分支 [^steemnova]

---

### 7. New-Star — 2Moons 重啟專案

| 屬性 | 內容 |
|------|------|
| **描述** | 基於 2Moons 的網頁遊戲引擎，目標以全新設計與玩法重啟開發。 |
| **GitHub** | <https://github.com/Yaro2709/New-Star> |
| **星數** | ⭐ 113 |
| **授權** | MIT |
| **狀態** | ❌ **已封存**（2025 年 3 月）— 最終版 v4.4.0 後專案已出售 |
| **語言** | PHP |

**主要功能：**
- 2Moons-based 引擎，設計全面翻新
- 外掛/模組系統
- 音效支援
- 專案已出售，不再活躍 [^new-star]

---

### 8. pr0game — 德國 OGame 克隆（營運中）

| 屬性 | 內容 |
|------|------|
| **描述** | 基於 SteemNova/2Moons 的德語 OGame 克隆。無付費取勝，由社群捐款營運。 |
| **GitHub (mirror)** | <https://github.com/X-Nightmare-X/pr0game> |
| **主要儲存庫** | <https://codeberg.org/pr0game/pr0game> |
| **授權** | AGPL-3.0 |
| **Live Game** | <https://pr0game.com> |
| **狀態** | ✅ **活躍** — 有營運伺服器與活躍社群 |
| **語言** | PHP |

**主要功能：**
- 基於 redesign 前的 OGame
- **無付費取勝** 機制
- 多種戰鬥引擎選項（SteemNova Array、DS、Julia）
- 活躍德語社群 [^pr0game]

---

### 9. Stellar Nomad — 太空探索遊戲（非直接克隆）

| 屬性 | 內容 |
|------|------|
| **描述** | 非 OGame 克隆，而是駕駛太空船、開採資源、探索隨機生成宇宙的開源瀏覽器遊戲。靈感來自 Bobiverse 書籍 / No Man's Sky。 |
| **GitHub** | <https://github.com/Resaki1/stellar-nomad> |
| **星數** | ⭐ 6 |
| **Live Demo** | <https://stellar-nomad.vercel.app> |
| **狀態** | 🟡 **初期開發階段** |
| **語言** | TypeScript / React Three Fiber / Next.js |

**主要功能：**
- 3D 太空探索（WASD 操控）
- 從小行星開採資源
- 飛船升級與製作
- 隨機生成宇宙（規劃中）
- 未來規劃多人模式
- ⚠️ 尚處於早期階段，與 OGame 規模無可比性 [^stellar-nomad]

---

## 對比總結

| 專案 | 多人 | 克隆準確度 | 現代技術 | 活躍度 | 最佳用途 |
|------|------|-----------|---------|--------|---------|
| **OGameX** | ✅ 是 | 很高 | ✅ Laravel + Rust | ✅ 非常活躍 | **最佳整體選擇** |
| **OGame Open Source** | ✅ 是 | **完美** (v0.84) | ❌ 經典 PHP | ✅ 活躍 | 懷舊 / 經典體驗 |
| **OGame Vue TS** | ❌ 否 (單機) | 高 | ✅ Vue 3 + TS | ✅ 非常活躍 | 離線練習 |
| **XG Proyect v4** | ✅ 是 | 高 | ✅ Laravel | ⚠️ 中等 | 自託管伺服器 |
| **pr0game** | ✅ 是 | 高 (pre-redesign) | ❌ PHP | ✅ 營運中 | 直接上線遊玩（德語） |
| **SteemNova** | ✅ 是 | 高 | ❌ PHP | ⚠️ 低 | 區塊鏈整合 |
| **2Moons** | ✅ 是 | 高 | ❌ PHP | ❌ 已封存 | 歷史參考 |
| **New-Star** | ✅ 是 | 高 | ❌ PHP | ❌ 已封存 | 歷史參考 |
| **Stellar Nomad** | ❌ 否 | 非克隆 | ✅ 現代 3D | 🟡 初期 | 太空探索（非 OGame 類似） |

## 推薦

1. **想玩或自託管現代 OGame** → **OGameX** 是最佳選擇：最活躍開發、現代技術棧、Live Demo、Rust 戰鬥引擎、完整功能。
2. **想自建私服** → OGameX（Docker-based）或 XG Proyect v4（Laravel）
3. **純粹懷舊** → OGame Open Source（v0.84，經典 2000 年代設計）
4. **離線沙盒練習** → OGame Vue TS（現代 UI，瀏覽器離線執行）

---

## 反思

本調查依賴 GitHub 專案頁面與開發者文件。部分專案星數較低（如 XG Proyect v4 僅 14 星、Stellar Nomad 僅 6 星），但因其在特定領域（分別為 Laravel OGame 框架與 3D 太空探索）的代表性而仍納入報告。讀者應自行評估專案品質與維護狀態。

---

[^ogamex]: lanedirt. (n.d.). *OGameX — Open-source OGame redesign clone built with Laravel*. GitHub. Retrieved 2026-09-10, from https://github.com/lanedirt/OGameX
[^ogame-opensource]: ogamespec. (n.d.). *ogame-opensource — Faithful recreation of OGame v0.84*. GitHub. Retrieved 2026-09-10, from https://github.com/ogamespec/ogame-opensource
[^ogame-vue-ts]: setube. (n.d.). *ogame-vue-ts — OGame tribute built with Vue 3, TypeScript and Vite*. GitHub. Retrieved 2026-09-10, from https://github.com/setube/ogame-vue-ts
[^xgproyect]: XG Proyect Team. (n.d.). *XGProyect — Open source OGame clone built with Laravel*. GitHub. Retrieved 2026-09-10, from https://github.com/XGProyect/XGProyect
[^2moons]: jkroepke. (n.d.). *2Moons — Open-source browsergame framework*. GitHub. Retrieved 2026-09-10, from https://github.com/jkroepke/2Moons
[^steemnova]: SteemNova Team. (n.d.). *steemnova — Multiplayer browser game with Steem blockchain*. GitHub. Retrieved 2026-09-10, from https://github.com/steemnova/steemnova
[^new-star]: Yaro2709. (n.d.). *New-Star — 2Moons restart project*. GitHub. Retrieved 2026-09-10, from https://github.com/Yaro2709/New-Star
[^pr0game]: X-Nightmare-X. (n.d.). *pr0game — German OGame clone*. GitHub. Retrieved 2026-09-10, from https://github.com/X-Nightmare-X/pr0game
[^stellar-nomad]: Resaki1. (n.d.). *stellar-nomad — Open source browser game about space exploration*. GitHub. Retrieved 2026-09-10, from https://github.com/Resaki1/stellar-nomad