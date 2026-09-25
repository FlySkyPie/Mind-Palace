# C++ 程式日誌記錄（Logging）實務調查

撰寫 C++ 程式時，記錄日誌（logging）是一項基礎但至關重要的基礎設施。本報告調查截至 2026 年 9 月，C++ 社群中最常見的日誌記錄方式、主流函式庫以及最佳實踐。

## 常見做法總覽

C++ 專案的日誌記錄大致可分為三類：

1. **使用專用日誌函式庫**（主流做法）—— 引入如 spdlog、quill 等第三方函式庫
2. **使用 Boost.Log** —— 若專案已依賴 Boost 生態系
3. **自製簡易日誌** —— 使用 `fstream`、`std::cout` 或預處理器巨集

## 主流日誌函式庫比較

### spdlog（最受歡迎，29.6k⭐）[^spdlog]

spdlog 是 C++ 日誌記錄的**事實標準**。它使用 `{fmt}` 格式化風格、支援標頭檔 only 或編譯模式、提供同步與非同步日誌、內建多種 sink（輸出目標，包括旋轉檔案、每日檔案、彩色終端機、syslog、Windows Event Log 等），並支援自定義格式器與 MDC（Mapped Diagnostic Context）。相容 C++11 以上，幾乎所有套件管理器皆可取得。

使用範例（語法層面）：

```cpp
#include <spdlog/spdlog.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <spdlog/sinks/rotating_file_sink.h>

// 基本用法
spdlog::info("Hello from spdlog! version {}", SPDLOG_VERSION);
spdlog::warn("Warning message");
spdlog::error("Error code: {}", 42);

// 設定日誌層級
spdlog::set_level(spdlog::level::debug);
spdlog::debug("Debug message"); // 僅在 debug 層級以上時輸出

// 自訂 pattern
spdlog::set_pattern("[%Y-%m-%d %H:%M:%S.%e] [%t] [%^%l%$] %v");

// 旋轉檔案 sink（依大小輪替）
auto logger = spdlog::rotating_logger_mt("file_logger", "logs/app.log",
                                         1048576 * 5, 3); // 5MB, 最多保留 3 個檔案

// 多 sink 組合：終端機顯示 WARN+，檔案記錄所有層級
auto console = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
console->set_level(spdlog::level::warn);
auto file = std::make_shared<spdlog::sinks::basic_file_sink_mt>("logs/all.log");
spdlog::logger multi_logger("multi", {console, file});
multi_logger.set_level(spdlog::level::trace);   // 最低層級由各 sink 獨立控制
```

**效能**：單執行緒中位數延遲約 271 ns，吞吐量約 257 萬筆/秒。在非同步模式下效能更佳。[^benchmark]

### quill（高效能新星，3.0k⭐）[^quill]

quill 是專為**極低延遲**與**高吞吐量**設計的 C++17 非同步日誌函式庫。前端（frontend）使用無鎖（lock-free）SPSC 佇列，格式化與 I/O 在背景執行緒進行，因此記錄端延遲僅約 **6 ns**（中位數）。內建 Prometheus metrics sink、JSON 輸出、MDC、崩潰處理、速率限制巨集與 huge pages 支援。[^benchmark]

適合對效能要求極高的場景，如遊戲引擎、高頻交易、即時系統。

### fmtlog（超低延遲，1.0k⭐）[^fmtlog]

fmtlog 也是非同步日誌函式庫，前端延遲同樣約 **6 ns**，特色是支援指標傳遞參數以避免複製大型物件，以及編譯時期日誌層級過濾與頻率限制。

### plog（極簡嵌入式，2.6k⭐）[^plog]

plog 是一款**極度輕量**（約 1000 行程式碼）、**無外部依賴**、支援 C++98 的標頭檔 only 日誌函式庫。非常適合嵌入式系統或資源受限的環境。

### g3log（崩潰安全）[^g3log]

g3log 的特色是**崩潰安全**：在 SIGSEGV、SIGABRT 等訊號發生時，會先 flush 佇列中日誌再終止程式，適合任務關鍵（mission-critical）系統。

### Boost.Log[^boostlog]

Boost.Log 是 Boost 生態系的完整日誌解決方案，提供階層式 logger、屬性系統、豐富的過濾機制與多種 sink。但效能顯著低於現代方案（約 **3093 ns** 單執行緒中位數延遲，吞吐量僅 33 萬筆/秒）。適合已重度依賴 Boost 的專案。

### 已不建議使用的函式庫

- **easyloggingpp**（3.9k⭐）—— 已於 2025 年 7 月封存（archived），官方建議遷移至 spdlog[^easyloggingpp]
- **Google glog**（7.4k⭐）—— 已於 2025 年 6 月封存，後繼者為 Abseil Logging / ng-log[^glog]
- **log4cpp / log4cxx** —— Log4j 風格的設定式日誌，屬於遺留方案

## 簡易自製日誌方式

### 方法一：檔案串流 + mutex

```cpp
#include <fstream>
#include <mutex>

class SimpleLogger {
    std::ofstream file_;
    std::mutex mtx_;
public:
    SimpleLogger(const std::string& path) {
        file_.open(path, std::ios::app);
    }
    template<typename T>
    SimpleLogger& operator<<(const T& val) {
        std::lock_guard<std::mutex> lock(mtx_);
        file_ << val;
        file_.flush();
        return *this;
    }
};
```

**優點**：零依賴、完全掌控。**缺點**：無日誌層級、無格式化 pattern、無輪替、I/O 阻塞、高競爭時效能差。

### 方法二：預處理器巨集

```cpp
#define LOG_INFO(msg) std::cout << "[INFO] " << __FILE__ \
    << ":" << __LINE__ << " - " << msg << std::endl

#ifdef NDEBUG
  #define LOG_DEBUG(msg) // 在 Release 模式編譯時展開為空
#else
  #define LOG_DEBUG(msg) std::cout << "[DEBUG] " << __FILE__ \
      << ":" << __LINE__ << " - " << msg << std::endl
#endif
```

**優點**：可於編譯時期移除 Debug 日誌、實作簡單。**缺點**：功能有限、無非同步、不具擴充性。

### 方法三：C++20 `std::format` + `std::source_location`

若使用 C++20/23，無外部依賴即可實現帶格式化與原始碼位置的簡易日誌。

## 效能基準

根據 quill v13.0.0 在 Intel i5-12600、GCC 14.2 上的基準測試：[^benchmark]

| 函式庫 | 單執行緒延遲（50%ile） | 4 執行緒延遲（50%ile） | 吞吐量（百萬筆/秒） |
|---|---|---|---|
| fmtlog | **6 ns** | **8 ns** | 2.69 |
| quill | **6 ns** | **8 ns** | **6.44** |
| spdlog | 271 ns | 557 ns | 2.57 |
| Boost.Log | 3,093 ns | 1,582 ns | 0.33 |

## 最佳實踐

### 架構設計

- **使用專用函式庫，不要自造輪子** —— spdlog、quill 已經過大規模驗證
- **有非同步就用非同步** —— 非同步日誌可將 I/O 延遲從應用程式執行緒分離
- **採用「前端/後端」模式** —— 前端僅將參數序列化至佇列，格式化和 I/O 由背景執行緒處理
- **設定日誌層級過濾** —— 編譯時期使用巨集（如 `SPDLOG_ACTIVE_LEVEL`）移除低層級日誌；執行時期使用 `set_level()` 動態調整[^spdlog]

### 生產環境

- **啟用日誌輪替** —— 基於檔案大小或時間，避免磁碟空間耗盡
- **定期 flush** —— spdlog 的 `flush_every(3s)` 可降低崩潰時遺失日誌的風險
- **考量崩潰處理** —— 使用 g3log 或 quill 的訊號捕獲機制，確保崩潰前日誌已寫入
- **使用結構化日誌** —— JSON 格式日誌便於 ELK、Loki、Datadog 等日誌聚合工具處理
- **為頻繁輸出的日誌設定速率限制** —— quill 的 `LOG_INFO_LIMIT` 和 fmtlog 的 `FMTLOG_LIMIT` 支援每呼叫點的頻率限制

### 程式碼風格

- **優先使用 `{fmt}` / `std::format` 風格** —— 型別安全、效能更佳、可讀性更高。避免 iostream `<<` 串接或 printf
- **使用 MDC 添加上下文** —— 自動在每行日誌中加入 request ID、session ID 或使用者 ID
- **不要在靜態解構子中記錄日誌** —— 函式庫內部 singleton 可能已被銷毀
- **使用一致的日誌 pattern** —— 建議格式：`[%Y-%m-%d %H:%M:%S.%e] [%t] [%l] %v`
- **記錄上下文而非純字串** —— 包含函式名稱、檔案/行號、關聯 ID

## 選擇指南

```
一般用途、希望社群支援最豐富 → spdlog（90% 專案的預設選擇）
需要極低延遲（<10 ns）       → quill（功能最豐富）或 fmtlog（延遲最低）
需要崩潰安全                  → g3log
需要極簡、零依賴、嵌入式      → plog
已重度使用 Boost              → Boost.Log
```

## 參考資料

[^spdlog]: Gabime. (n.d.). spdlog: Fast C++ logging library. Retrieved 2026-09-25, from https://github.com/gabime/spdlog

[^quill]: Odygrd. (n.d.). Quill: Asynchronous low-latency logging library for C++. Retrieved 2026-09-25, from https://github.com/odygrd/quill

[^fmtlog]: MengRao. (n.d.). fmtlog: A performant fmt-style C++ logging library. Retrieved 2026-09-25, from https://github.com/MengRao/fmtlog

[^plog]: SergiusTheBest. (n.d.). plog: Portable, simple and extensible C++ logging library. Retrieved 2026-09-25, from https://github.com/SergiusTheBest/plog

[^g3log]: KjellKod. (n.d.). g3log: Asynchronous, crash-safe logging library for C++. Retrieved 2026-09-25, from https://github.com/KjellKod/g3log

[^glog]: Google. (n.d.). glog: C++ implementation of the Google logging module. Retrieved 2026-09-25, from https://github.com/google/glog

[^easyloggingpp]: eCash Information. (n.d.). Easylogging++: C++ logging library. Retrieved 2026-09-25, from https://github.com/abumq/easyloggingpp

[^boostlog]: Boost Organization. (n.d.). Boost.Log. Retrieved 2026-09-25, from https://github.com/boostorg/log

[^benchmark]: Odygrd. (n.d.). Quill v13.0.0 Benchmarks. Retrieved 2026-09-25, from https://github.com/odygrd/quill#benchmarks