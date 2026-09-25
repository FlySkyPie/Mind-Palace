# C++ 非同步檔案 I/O 支援分析

## 標準 C++ 有非同步檔案 I/O 嗎？

**沒有。** C++11 至 C++26 的標準函式庫中，**沒有任何原生的非同步或非阻塞檔案讀寫能力**。標準函式庫缺少讓檔案 `read()` 或 `write()` 以非阻塞方式執行的設施，也沒有在任何 I/O 完成時進行非同步等待的機制。

## 標準函式庫提供的內容（以及缺乏的內容）

### `std::fstream` / `std::ifstream` / `std::ofstream`（C++98 起，`<fstream>`）

- 全部是**同步且阻塞**的。每次讀寫操作都會阻塞呼叫的執行緒，直到 I/O 在作業系統層級完成[^fstream-block]。
- 無法在 `std::fstream` 上設定 `O_NONBLOCK`。標準不暴露底層的檔案描述符，`std::filebuf::open()` 也沒有對應作業系統層的 `O_NONBLOCK`、`O_SYNC` 或 `O_DIRECT` 等標誌[^fstream-nonblock]。

### `std::async` / `std::future`（C++11，`<future>`）

- `std::async` 在另一個執行緒上執行可呼叫物件，但這是**執行緒層級的並行**，不是非同步 I/O[^async-vs-aio]。
- 底層的 `read()`/`write()` 系統呼叫在核心層級仍然是阻塞的 — 只不過是在不同執行緒上阻塞。
- P2300 提案本身指出：`std::async`/`std::future`/`std::promise` 作為 C++11 的非同步機制，「效率低、難以正確使用、且嚴重缺乏通用性」[^p2300]。

### `std::filesystem`（C++17，`<filesystem>`）

- **全部是同步的。** `copy()`、`remove()`、`create_directory()` 等操作均為阻塞，沒有非同步的變體[^filesystem-block]。

### C++20 協程（`co_await`，`<coroutine>`）

- C++20 新增了**語言層的協程機制**（無堆疊、編譯器生成狀態機）。協程只是編寫暫停/恢復程式碼的方式 — **它們本身不提供任何 I/O**[^coro-no-io]。
- 你需要一個非同步 I/O 函式庫（如 Boost.Asio 或自訂的 io_uring 封裝）來提供 `co_await` 可以掛起的 awaitable 物件。

## C++26 `std::execution`（P2300）— Sender/Receiver 模型

P2300R10 已被 C++26 接納，提供了 `<execution>` 標頭檔，包含 sender/receiver/scheduler 模型[^p2300-adopted]。

### 它是什麼：
- 一個**組合非同步操作的框架**：`schedule()`、`then()`、`when_all()`、`let_value()`、`upon_error()` 等。
- `std::execution::run_loop` — 一個基本的執行上下文。
- 透過 `std::stop_token` 提供取消功能。

### 它不是什麼：
- **它不包含檔案 I/O 操作。** C++26 標準中沒有 `read_file()` 或 `write_file()` 的 sender。該提案將檔案 I/O 列為動機之一，但並未提供其實現[^p2300-no-fileio]。
- P2300 提供了**管道**（scheduler、sender、receiver、演算法），但沒有**水龍頭**（實際的檔案/網路 I/O 操作）。

## 從未進入標準的提案

### P1031 — Low-Level File I/O Library（LLFIO / AFIO）

- 由 Niall Douglas 撰寫，提案了一個零複製、低階的非同步檔案 I/O 函式庫[^p1031]。
- 提供 `file_handle`、`async_file_handle`、`map_handle` 等。
- **從未被採用。** 該專案作為開源函式庫 [ned14/llfio](https://github.com/ned14/llfio) 繼續發展。

### Networking TS（技術規格書）

- 最初基於 ASIO，包含網路 socket 和基本 I/O，但從未被完整標準化。
- 目前 C++ 委員會的立場似乎是：讓 `std::execution` 提供框架，將實際的 I/O 操作留給函式庫和平台 API[^net-ts]。

## 平台特定的非同步檔案 I/O

### Linux：`io_uring`（Linux 5.1+，現代選擇）

- **真正的核心層級非同步檔案 I/O**，使用使用者空間與核心之間的共享環形緩衝區（提交佇列 / 完成佇列）[^iouring]。
- 支援**緩衝與非緩衝**（O_DIRECT）I/O — 與舊版 API 不同。
- 消除系統呼叫開銷：提交和完成透過記憶體映射環完成。
- 支援 socket、檔案、管道、計時器等。
- 可達到 248,000 IOPS（QD=32，4KB 隨機讀取），而 libaio 為 142,000，POSIX AIO 為 28,500[^iouring-perf]。
- Boost.Asio 在 Linux 上的檔案操作後端（當定義了 `BOOST_ASIO_HAS_IO_URING` 時）。

### Linux：`libaio`（Linux 2.6+，舊版）

- 透過 `io_submit()`/`io_getevents()` 進行的直接核心 AIO。
- **只能與 O_DIRECT 搭配使用（非緩衝）**。緩衝 I/O 會退回為同步。
- 需要對齊緩衝區（通常為 512 位元組）[^libaio]。
- 正被 io_uring 取代。

### Linux/POSIX：POSIX AIO（`aio_read`、`aio_write`）

- 跨平台 API（POSIX.1b）。
- **在 Linux 上，glibc 使用使用者空間執行緒池實作此功能** — 每個 `aio_read()` 會產生一個執行緒來同步呼叫 `pread()`。並非真正的核心層級非同步[^posix-aio-linux]。
- 無法擴展：每個操作消耗約 8MB 執行緒堆疊，限制在約 100 個並發操作。

### Windows：IOCP（I/O Completion Ports）

- **真正的核心層級非同步檔案 I/O**，透過重疊 I/O（`ReadFile`/`WriteFile` 搭配 `OVERLAPPED` 結構 + `CreateIoCompletionPort`）[^iocp]。
- Proactor 模型：提交帶有緩衝區的讀取操作，在完成時收到通知。
- Boost.Asio 在 Windows 上的主要後端。

### BSD/macOS：`kqueue`

- **kqueue 不為一般檔案提供真正的非同步檔案 I/O。** 一般檔案的檔案描述符永遠報告為「就緒」可讀寫，因此無法進行非同步完成通知[^kqueue-file]。
- macOS 的 Grand Central Dispatch（GCD）提供 `dispatch_read()`/`dispatch_write()` 來進行非同步檔案 I/O，底層使用 kqueue 加執行緒池。

### `mmap`（記憶體映射檔案）

- 將檔案內容映射到行程的虛擬位址空間。透過記憶體讀寫進行存取。
- 核心透明地處理頁面錯誤 — 當你存取尚未載入 RAM 的頁面時，執行緒會阻塞直到頁面載入完成[^mmap-pagefault]。
- **並非真正的非同步** — 頁面錯誤會造成你無法控制或排程的阻塞。在基於協程的程式碼中被視為「非同步危險」。

| 功能 | 非同步檔案 I/O？ | 備註 |
|---|---|---|
| `std::fstream`/`ifstream`/`ofstream` | ❌ 否 | 總是阻塞，無非阻塞模式 |
| `std::async` + `std::future` | ❌ 否 | 執行緒層級，非非同步 I/O |
| `std::filesystem` | ❌ 否 | 全部同步 |
| C++20 協程（`co_await`） | ⚠️ 僅機制 | 需要函式庫提供實際 I/O |
| C++26 `std::execution`（P2300） | ⚠️ 僅框架 | Sender/receiver 管道，無檔案 I/O sender |
| P1031（LLFIO/AFIO） | ❌ 未標準化 | 提案被拒，以開源形式存在 |
| **Boost.Asio**（`stream_file`/`random_access_file`） | ✅ 是 | Linux 上用 io_uring，Windows 上用 IOCP |
| **Linux io_uring**（liburing） | ✅ 是 | 現代、高效能、支援緩衝 + 直接模式 |
| **Linux libaio** | ⚠️ 部分 | 僅 O_DIRECT，舊版 |
| **POSIX AIO**（`aio_read`） | ❌ 非真正非同步 | glibc 中為執行緒池實作 |
| **Windows IOCP** | ✅ 是 | 重疊 I/O，經生產驗證 |
| **BSD/macOS kqueue**（對檔案） | ❌ 否 | 檔案永遠「就緒」，無非同步完成 |
| **mmap** | ❌ 否 | 頁面錯誤會透明地阻塞 |

## 結論

C++26 標準函式庫仍然沒有原生的非同步檔案 I/O 支援。若需要非同步檔案讀寫，開發者必須使用平台特定 API（Linux 上的 io_uring、Windows 上的 IOCP）或第三方函式庫（Boost.Asio 提供統一的跨平台非同步檔案 I/O 介面）。C++26 新增了 `std::execution` 作為非同步組合的框架，但並未包含實際的檔案 I/O 操作。

[^fstream-block]: cppreference.com. (n.d.). `std::basic_fstream`. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/io/basic_fstream

[^fstream-nonblock]: Stack Overflow. (n.d.). Non-blocking write to file in C/C++. Retrieved 2026-09-25, from https://stackoverflow.com/questions/4434223/non-blocking-write-to-file-in-c-cpp

[^async-vs-aio]: Petrovic, D. (n.d.). C++ Executors: From `std::async` to `std::execution`. Retrieved 2026-09-25, from https://daniel-petrovic.github.io/blog/cpp-executors/

[^p2300]: Open Standards. (2024). P2300R10: `std::execution`. Retrieved 2026-09-25, from https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2300r10.html

[^filesystem-block]: cppreference.com. (n.d.). `std::filesystem`. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/filesystem

[^coro-no-io]: Stack Overflow. (n.d.). True non-threaded asynchronous file I/O in C++. Retrieved 2026-09-25, from https://stackoverflow.com/questions/79723888/true-non-threaded-asynchronous-file-io-in-cpp

[^p2300-adopted]: cppreference.com. (n.d.). C++ Execution control library. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/execution

[^p2300-no-fileio]: Stack Overflow. (n.d.). How is the new asynchronous model in C++26 different from existing models?. Retrieved 2026-09-25, from https://stackoverflow.com/questions/78829184/how-is-the-new-asynchronous-model-in-c26-different-from-existing-models

[^p1031]: Open Standards. (2019). P1031R2: Low-Level File I/O Library. Retrieved 2026-09-25, from https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1031r2.pdf

[^net-ts]: Think Async. (n.d.). Boost.Asio Overivew: Files. Retrieved 2026-09-25, from https://think-async.com/Asio/boost_asio_1_28_0/doc/html/boost_asio/overview/files.html

[^iouring]: Cor3ntin. (n.d.). io_uring by example. Retrieved 2026-09-25, from https://cor3ntin.github.io/posts/iouring/

[^iouring-perf]: PiStack. (2026). Linux Async IO: libaio, POSIX AIO, io_uring. Retrieved 2026-09-25, from https://www.pistack.xyz/posts/2026-06-02-linux-async-io-libaio-posix-aio-iouring-guide/

[^libaio]: man7.org. (n.d.). aio(7) — POSIX asynchronous I/O overview. Retrieved 2026-09-25, from https://www.man7.org/linux/man-pages/man7/aio.7.html

[^posix-aio-linux]: Stack Overflow. (n.d.). Asynchronous I/O versus memory-mapped file I/O. Retrieved 2026-09-25, from https://stackoverflow.com/questions/79694779/asynchronous-io-versus-memory-mapped-file-io

[^iocp]: Boost.Asio Documentation. (n.d.). Overview: Files. Retrieved 2026-09-25, from https://www.boost.org/doc/libs/latest/doc/html/boost_asio/overview/files.html

[^kqueue-file]: Stack Overflow. (n.d.). Using kqueue for simple async I/O. Retrieved 2026-09-25, from https://stackoverflow.com/questions/35236247/using-kqueue-for-simple-async-io

[^mmap-pagefault]: Áron, Ó. (n.d.). Async I/O Explained. Retrieved 2026-09-25, from https://olafuraron.is/blog/async-io-explained/