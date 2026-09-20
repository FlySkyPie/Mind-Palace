# C++ HTTP Server 函式庫比較研究

## 概述

C++ 作為系統級程式語言，擁有多種用於建立 HTTP 伺服器的開源函式庫。本報告針對 2025–2026 年間活躍維護的主流選項進行調查與比較，涵蓋從全功能框架到低階協定實作的完整光譜。

## 主要函式庫一覽

| 函式庫 | GitHub Stars | 最新版本 | C++ 標準 | 授權 |
|---|---|---|---|---|
| **uWebSockets** | ~19.0k | v20.80.0 (2026-09-03) | C++17/20 | Apache 2.0 |
| **Drogon** | ~14.3k | v1.9.13 (2026-05-07) | C++14/17/20 | MIT |
| **Oat++ (oatpp)** | ~8.7k | 1.3.1 (2025-12-01) | C++11+ | Apache 2.0 |
| **Crow** | ~5.0k | v1.3.4 (2026-09-07) | C++11/14 | BSD-3 |
| **Beast (Boost)** | ~4.8k | 隨 Boost 發行 | C++11 | BSL-1.0 |
| **Pistache** | ~3.5k | v0.4.26 (2024-12-23) | C++17 | Apache 2.0 |
| **restbed** | ~2.0k | 5.0.0 LTS (2026-04-06) | C++23 **強制** | AGPL / 商業 |
| **CPPRestSDK** | ~8.2k | v2.10.19 (2025-12-05) | C++11 | MIT ⚠️ **已封存** |

## 詳細分析

### 1. uWebSockets

最受歡迎的 C++ HTTP 伺服器引擎，擁有最高的 GitHub 星數。

- **特點**：極高效能 HTTP + WebSocket 伺服器；TLS 1.3 速度超越多數純文字伺服器；內建 URL 路由（支援萬用字元與參數）；WebSocket pub/sub 機制；底層基於 µSockets（支援 libuv/ASIO/GCD/epoll/kqueue）。[^uwebsockets]
- **優勢**：效能頂尖；安全紀錄優異（10 年間僅 2 個 CVE）；由大型加密貨幣交易所生產環境驗證；OSS-Fuzz 每日覆蓋率約 95%。[^uwebsockets]
- **劣勢**：核心為 C 語言 + C++ 包裝層，C++ 語法風格較不純粹；無 ORM/資料庫整合；Windows 支援有限。[^uwebsockets]
- **適用場景**：需要極致效能的即時應用（如加密貨幣交易所、金融交易系統、即時通訊）。

### 2. Drogon

功能最全面的 C++ HTTP 應用框架。

- **特點**：基於 epoll/kqueue 的非阻塞 I/O；完整非同步模型；HTTP 1.0/1.1（伺服器 + 用戶端）；模板反射解耦控制器與視圖；Cookie 與內建 Session；動態視圖載入；靈活路由與過濾鏈；HTTPS（OpenSSL）；WebSocket（伺服器 + 用戶端）；JSON 請/回應；檔案上傳/下載；gzip/brotli 壓縮；pipelining；CLI 工具；非同步資料庫支援（PostgreSQL、MySQL、SQLite3、Redis）；輕量 ORM；外掛系統；AOP；C++ Coroutine；HTTP/2 用戶端（v1.10 beta）。[^drogon]
- **優勢**：TechEmpower 基準測試中名列前茅；功能集最完整；開發活躍；文件完善。[^drogon]
- **劣势**：巨集設定較多，學習曲線較陡；比微框架更重。[^drogon]
- **適用場景**：需要資料庫整合、ORM、高效能的完整 Web 應用。

### 3. Oat++ (oatpp)

零依賴的輕量 Web 框架。

- **特點**：完全零外部依賴；高度可移植；REST API 搭配 Swagger-UI 自動文件；ORM（SQLite、PostgreSQL、MongoDB）；WebSocket；Simple 與 Async 兩種 API；HLS 串流；TLS；連線池；二進位大小約 1MB。[^oatpp]
- **優勢**：真正的零依賴；適合 IoT/嵌入式系統；豐富的生態模組；內建 Swagger-UI 整合。[^oatpp]
- **劣勢**：效能不及 Drogon 與 uWebSockets；不適合極低延遲場景。[^oatpp]
- **適用場景**：IoT/嵌入式裝置、需要 Swagger 文件的 REST API、零依賴需求的專案。

### 4. Crow

仿 Flask 風格的極簡微框架。

- **特點**：Flask 風格路由；編譯期型別安全處理器；header-only（可選單一標頭檔）；內建 JSON 支援；Mustache 模板；中介層支援；HTTP/1.1 + WebSocket；multipart 請/回應；ASIO/Boost.ASIO 後端。[^crow]
- **優勢**：API 極簡易用；整合門檻最低（header-only）；效能表現優異。[^crow]
- **劣勢**：尚無 HTTP/2 支援（規劃中）；非同步支援仍在開發；功能不如 Drogon/Oat++ 豐富。[^crow]
- **適用場景**：快速原型開發、小型內部工具、偏好 Flask 風格 API 的開發者。

### 5. Beast (Boost.Asio)

Boost 官方 HTTP/WebSocket 低階實作。

- **特點**：header-only；HTTP/1 與 WebSocket 協定詞彙型別；建構於 Boost.Asio 的一致性非同步模型；角色中立（同時支援用戶端與伺服器）；OpenSSL TLS；是高階函式庫的基礎建構塊。[^beast]
- **優勢**：Boost 生態的一員，經過大規模實戰驗證；header-only；文件詳盡；非常穩健。[^beast]
- **劣勢**：低階 API（無路由器、無框架功能）；需大量樣板程式碼；依賴 Boost。[^beast]
- **適用場景**：建立自訂網路協定、需要 Boost.Asio 整合、作為更高階函式庫的底層基礎。

### 6. Pistache

注重 API 優雅的 REST 工具包。

- **特點**：多執行緒 HTTP 伺服器；非同步 HTTP 用戶端；路由分配至 C++ 函式；REST 描述 DSL；型別安全的標頭與 MIME；SSL；libevent 後端（macOS/BSD/Windows）；epoll（Linux）；Brotli、deflate、zstd 內容編碼。[^pistache]
- **優勢**：API 設計簡潔現代；優雅的 REST DSL；官方文件網站良好。[^pistache]
- **劣勢**：仍為 1.0 以下版本（不穩定）；HTTP 用戶端已知有問題；發版頻率較低；社群較小。[^pistache]
- **適用場景**：偏好優雅語法的小型 REST 服務。

### 7. restbed

企業級非同步 REST 框架。

- **特點**：企業級非同步 RESTful 框架；WebSocket；Server-Sent Events；Comet/long-polling；SSL/TLS；HTTP pipelining；路徑/查詢參數；多路徑資源；自訂 HTTP 方法；壓縮；IPv4/IPv6；認證；訊號處理。[^restbed]
- **優勢**：企業級支援可選商業授權；長期支援（重大版本維護 10+ 年）。[^restbed]
- **劣勢**：**強制要求 C++23**（需 GCC ≥ 13、Clang ≥ 17）；v5.0 有重大破壞性變更；HTTP 用戶端於 v5.0 移除；使用 GNU Autotools（非 CMake）；社群較小。[^restbed]
- **適用場景**：已採用 C++23 的企業專案、需要商業支援的組織。

### 8. CPPRestSDK (Casablanca) — ⚠️ 已封存

微軟主導的 C++ REST SDK。

- **特點**：HTTP 用戶端/伺服器；JSON；URI；非同步串流（PPL Tasks）；WebSocket 用戶端；OAuth；跨平台。[^cpprestsdk]
- **現狀**：**此儲存庫已於 2026 年 6 月 1 日封存**，不再維護。微軟官方建議轉移至 **libcurl** 或 **Boost.Beast**。[^cpprestsdk]
- **結論**：不應在新專案中使用。

## 綜合比較

| 面向 | uWebSockets | Drogon | Oat++ | Crow | Beast | Pistache | restbed |
|---|---|---|---|---|---|---|---|
| **抽象層級** | 伺服器引擎 | 全框架 | 全框架 | 微框架 | 低階函式庫 | REST 工具包 | 全框架 |
| **HTTP/2** | ❌ | Beta 用戶端 | ❌ | 規劃中 | ❌ | ❌ | ❌ |
| **WebSocket** | ✅ 伺服+用戶 | ✅ 伺服+用戶 | ✅ | ✅ | ✅ 伺服+用戶 | ❌ | ✅ |
| **ORM/資料庫** | ❌ | ✅ PG/MySQL/SQLite/Redis | ✅ PG/SQLite/Mongo | ❌ | ❌ | ❌ | ❌ |
| **Header-only** | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ |
| **非同步 API** | ✅ | ✅ 完整 | ✅ | 有限 | ✅ (Asio) | ✅ | ✅ |
| **C++ Coroutine** | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **活躍維護** | ✅ | ✅ | ✅ | ✅ | ✅ (Boost) | ✅ | ✅ |

## 選擇建議

根據不同需求推薦：

1. **極致效能、即時應用** → **uWebSockets**（星數最高、效能最佳、安全紀錄優異）
2. **完整 Web 應用、資料庫整合** → **Drogon**（功能最全面、效能頂尖、支援 ORM/DB/Coroutine）
3. **IoT/嵌入式系統、零依賴** → **Oat++**（零外部依賴、體積小、Swagger 整合）
4. **快速原型、小型專案** → **Crow**（Flask 風格、header-only、整合容易）
5. **自訂低階協定、Boost 生態** → **Beast**（Boost 官方、高可靠、低階控制）
6. **優雅 REST DSL** → **Pistache**（API 簡潔、但注意仍為 pre-1.0）
7. **企業 C++23 專案** → **restbed**（C++23 強制、商業授權選項）
8. **不建議新專案使用** → **CPPRestSDK**（已封存）

## 參考資料

[^uwebsockets]: uNetworking. (n.d.). uWebSockets — Simple, secure & standards compliant web server. Retrieved 2026-09-18, from https://github.com/uNetworking/uWebSockets
[^drogon]: Drogon Framework. (n.d.). Drogon — A C++14/17/20 based HTTP web application framework. Retrieved 2026-09-18, from https://github.com/drogonframework/drogon
[^oatpp]: oatpp. (n.d.). Oat++ — Light and powerful C++ web framework for highly scalable and resource-efficient web application. Retrieved 2026-09-18, from https://github.com/oatpp/oatpp
[^crow]: CrowCpp. (n.d.). Crow — A Fast and Easy to use microframework for the web. Retrieved 2026-09-18, from https://github.com/CrowCpp/Crow
[^beast]: Boost.org. (n.d.). Beast — HTTP and WebSocket built on Boost.Asio in C++11. Retrieved 2026-09-18, from https://github.com/boostorg/beast
[^pistache]: Pistache. (n.d.). Pistache — A high-performance REST toolkit written in C++. Retrieved 2026-09-18, from https://github.com/pistacheio/pistache
[^restbed]: Corvusoft. (n.d.). restbed — Corvusoft's Restbed framework brings asynchronous RESTful functionality to C++ applications. Retrieved 2026-09-18, from https://github.com/Corvusoft/restbed
[^cpprestsdk]: Microsoft. (n.d.). cpprestsdk — The C++ REST SDK is a Microsoft project for cloud-based client-server communication in native code using a modern asynchronous C++ API design. Retrieved 2026-09-18, from https://github.com/microsoft/cpprestsdk