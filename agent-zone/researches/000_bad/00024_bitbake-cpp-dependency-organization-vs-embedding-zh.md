# BitBake 用於 C++ 依賴組織以取代 Embedding 的可行性分析

> [!WARNING] 對齊失敗
> "instead of Embedding" 是指"不將 BitBack 用於嵌入式開發。而是單純處理套件仰賴問題"，不是指"將第三方套件嵌入在專案中"

## 摘要

本報告探討能否使用 Yocto Project 的 BitBake 來組織一般 C++ 專案中的第三方函式庫依賴，以取代傳統的 embedding（vendoring，將原始碼直接放入專案倉庫）做法。結論是：技術上可行但高度不建議。BitBake 的 DEPENDS 機制與 SRC_URI 確實具備依賴管理所需的基本能力，但其設計目標是完整嵌入式 Linux 發行版建置，而非單一應用程式的第三方函式庫管理。對於一般 C++ 專案，Conan、vcpkg、CMake FetchContent 或 CPM.cmake 是更合適的選擇。

## 1. 問題背景：Embedding vs 外部依賴管理

C++ 專案在處理第三方函式庫時，傳統上常見兩種做法：

- **Embedding（vendoring）**：將第三方函式庫的原始碼直接複製一份到專案倉庫中（例如 `third_party/` 目錄），與專案原始碼一起版本控制、一起編譯。
- **外部依賴管理**：透過套件管理器或建置系統機制，在編譯過程中自動下載、建置、連結外部函式庫。

Embedding 雖然簡單直接，但存在顯著問題：無法自動取得上游更新、難以處理 Diamond dependency（A→C v1、B→C v2 的版本衝突）、倉庫膨脹、供應鏈安全難以追蹤[^cpp-dep-org]。

## 2. BitBake 的依賴管理機制

### 2.1 DEPENDS 與 RDEPENDS

BitBake 提供完整的依賴宣告機制[^bb-docs-intro]：

```bitbake
# 建置時期依賴（在當前配方之前建置並提供）
DEPENDS = "zlib libpng"

# 執行時期依賴（目標系統上必須安裝的套件）
RDEPENDS:${PN} = "libssl libcrypto"
```

BitBake 會根據所有配方的 DEPENDS 宣告，建構完整的**任務層級有向無環圖（DAG）**，自動決定任務執行順序與平行化策略。這在技術上確實達成了依賴管理的核心功能——自動解析、排序、並行建置。

### 2.2 SRC_URI：原始碼擷取

運算式可從多種來源自動取得原始碼[^bb-docs-intro]：

```bitbake
SRC_URI = "git://github.com/openssl/openssl.git;branch=openssl-3.0;protocol=https"
SRCREV = "fd16ebc5d4c1c0f1d7c5c0d1f1c5d0d1f1c5d0d1"
```

這使得 BitBake 不需要預先 embedding 原始碼——它可以在建置時自動從 GitHub、GitLab、HTTP、SVN 等來源擷取指定版本的程式碼。此機制與 CMake FetchContent 或 CPM.cmake 的概念類似。

### 2.3 圖層系統與 bbappend 覆寫

BitBake 的圖層（Layer）系統允許將不同關注點分離至不同目錄。`.bbappend` 檔案可以在不修改原始配方的情況下，新增或覆寫任務，這是一種乾淨的依賴客製化機制——類似在不安裝第三方套件的 patch 版的情況下覆寫特定行為[^bb-docs-intro]。

## 3. BitBake 與 Embedding 的替代比較

| 面向 | Embedding（vendoring） | BitBake 管理依賴 |
|------|----------------------|------------------|
| 原始碼位置 | 在專案倉庫內 `third_party/` | 在 SRC_URI 指定的遠端倉庫 |
| 版本控制 | 隨專案一起 commit | SRCREV 精確鎖定版本 |
| 供應鏈追蹤 | 難以自動化 | 透過 checksum（SRC_URI[sha256sum]）驗證 |
| 上遊更新 | 手動複製 | 修改 SRCREV + PR |
| 建制隔離 | 與主專案共用建制環境 | 每配方獨立工作目錄 |
| 跨平臺 | 無特殊支援 | Linux only |
| 遞迴依賴 | 手動管理或子模組 | DEPENDS + RDEPENDS DAG 自動解析 |
| 貢獻上游 | 容易（原始碼已在 repo 中） | 困難（需從配方中提取 patch） |

## 4. 使用 BitBake 管理依賴的實際案例

### 4.1 官方 BitBake Hello World

Yocto 文件的 BitBake 手冊第 9 章展示了一個獨立於 Yocto 環境之外的 Hello World 範例，直接使用 `do_compile` 任務編譯 C 程式。該範例使用 BitBake 內建的 `base.bbclass`（其中定義了預設的 `do_compile` 與 `do_install`）來建置一個 C 原始碼檔案——展示 BitBake 可以獨立執行軟體建置任務[^bb-hello]。

### 4.2 a4z 的獨立 BitBake 指南

一份完全沒有 Yocto 的獨立 BitBake 教學，展示從最小專案開始，逐步加入配方、類別、圖層、以及 `.bbappend` 覆寫機制。該指南示範如何使用 `DEPENDS` 讓一個配方依賴另一個配方，並使用 `bitbake-layers` 管理圖層之間的關係[^bb-guide]。

### 4.3 BitBake 開發郵件列表討論（2023）

有開發者詢問是否可將 BitBake 用作獨立的套件管理器（替代 Conan 或 vcpkg）[^mailing]。討論中的反饋普遍指出：

- BitBake 的任務執行引擎確實可以處理套件的擷取、建置、安裝流程
- 但缺乏集中的套件倉庫或套件發現機制
- 沒有類似 Conan Center 或 vcpkg ports 的生態系
- 用來管理「主機系統上為該主機系統原生編譯的軟體依賴」並非其最佳化目標

## 5. BitBake 用於這類場景的具體優勢

1. **精確的可重現性**：BitBake 的簽章系統（Signature-based build）可以追蹤每一個任務的輸入變數。如果輸入未變更，輸出可安全快取。這類似 Bazel 的 hermetic build 概念，遠比 embedding 時依賴開發者手動管理更可靠[^bb-docs-intro]。

2. **DAG 層級的平行化**：當專案有多個無相互依賴的第三方函式庫時，BitBake 可以自動最大化平行建置效率——這在 embedding 時需要開發者手動安排建置順序。

3. **原始碼隔離**：每個 BitBake 配方的工作目錄都獨立於主專案，可以避免 embedding 時常見的 include path 污染或巨集衝突[^bb-guide]。

4. **層級覆寫**：`.bbappend` 允許為特定函式庫新增 patch 或修改組態，而無需 fork 維護一個修補版的 repository——這對 embedding 場景需要維護 fork 時來說是一個優勢。

## 6. BitBake 用於此場景的重大限制

### 6.1 無套件生態系

與 Conan（Conan Center 有超過 2000 個套件）或 vcpkg（超過 2000 個 port）不同，BitBake 沒有專為一般 C++ 應用程式設計的第三方函式庫套件倉庫。Yocto/OpenEmbedded 雖然有大量配方，但那些配方都是為了嵌入式 Linux 目標環境設計的，不是為桌面主機環境設計。

開發者若想用 BitBake 管理依賴，必須：
- 為**每一個**第三方函式庫從頭撰寫 `.bb` 配方
- 確保配方在目標主機系統上正確運作（非交叉編譯）
- 維護配方與上游版本的同步
- 處理每個函式庫獨特的建置系統（CMake、Make、Meson 等）

### 6.2 不支援 CMake 層級的依賴傳遞

BitBake 的 DEPENDS 是**配方（package）層級**的依賴，而非 CMake **target** 層級。這意味著：
- 如果函式庫 A 依賴函式庫 B，DEPENDS 只會確保 B 在建置 A 之前被建置好
- 它不會自動傳遞 B 的 include 路徑、編譯標誌、或連結庫給 A 的使用者
- 開發者需要在配方中手動管理 `CFLAGS`、`LDFLAGS`、`-I` 等

### 6.3 Linux only 且學習曲線極高

| 限制 | 說明 |
|------|------|
| 僅限 Linux | BitBake 無法在 Windows 或 macOS 上原生運行 |
| Python 3.9+ 依賴 | 需要正確設定 PATH 與 PYTHONPATH |
| 陡峭的學習曲線 | 即使是最小專案也需理解 BBPATH、BBLAYERS、配方命名、任務旗標等概念 |
| 文件稀少的獨立使用文件 | 絕大多數 BitBake 文件假設 Yocto/OpenEmbedded 環境 |

### 6.4 傳統 embedding 的優勢喪失

Embedding 最大的優勢之一是：**開發者可以直接修改第三方原始碼進行除錯或實驗**。用 BitBake 管理後，開發流程變成「修改配方 → 重新擷取 → 重新編譯」，失去了 embedding 時「直接在 IDE 中跳入第三方原始碼」的便利性。

## 7. 更適合的替代方案

對於一般 C++ 專案中「不要 embedding、用外部依賴管理」的需求：

| 工具 | 適用場景 | 與 Embedding 的比較 |
|------|---------|---------------------|
| **CMake FetchContent** | 零外部工具依賴，CMake 3.11+ 內建 | 類比：語法簡單，但每次從原始碼編譯，無中央快取 |
| **CPM.cmake** | FetchContent 封裝，加入版本鎖定與區域快取 | 類比：相較於 embedding 提供版本釘選，但無套件生態系 |
| **Conan** | 完整套件管理器，支援預編譯二進位 | 最佳替代：有 Conan Center 生態系、CMake 整合、跨平台 |
| **vcpkg** | Microsoft 維護，Manifest mode 為主流 | 最佳替代：超過 2000 port、CMake toolchain 整合、跨平台 |
| **Bazel / Pants** | 需要 hermetic build 與大規模 monorepo | 類比：學習曲線也高，但專為一般軟體建置設計 |

在 **00021** 與 **00022** 這兩篇報告中，已經詳細比較了這些選項的優劣，結論一致：CMake + Conan/vcpkg 或 CPM.cmake 是管理一般 C++ 第三方函式庫依賴的最佳路徑。

## 8. 結論

BitBake 的 **DEPENDS + SRC_URI + DAG 任務引擎** 在技術層面上的確具備了取代 embedding 所需的核心功能——自動擷取、版本鎖定、依賴排序、平行建置、簽章快取。獨立於 Yocto 之外也存在運作範例。

然而，**強烈不建議**將 BitBake 用於一般 C++ 專案的依賴組織來取代 embedding，原因如下：

1. **無套件生態系**：必須為每個函式庫從零撰寫配方，失去 embedding 的便利性卻無法獲得 Conan/vcpkg 的生態系好處。
2. **CMake 整合缺失**：依賴管理停留在「套件」層級而非「target」層級，無法自動傳遞編譯與連結資訊。
3. **平台限制**：Linux only，不支援跨平台開發。
4. **學習曲線與維護成本**：為依賴管理引入 BitBake 的複雜度遠遠超過問題本身。

如果你已經在使用 **Yocto Project 建置嵌入式 Linux 系統**，BitBake 很適合管理系統中的 C++ 元件。但如果你只是在開發一般桌面應用程式，需要一個替代 embedding 的依賴管理方案，請選擇 Conan、vcpkg 或 CPM.cmake——它們才是為這個問題設計的工具。

## 參考資料

[^cpp-dep-org]: FlyPie Mind Palace. (n.d.). *C++ 依賴管理與組織方式研究*. Retrieved 2026-09-13, from agent-zone/researches/00021_cpp-dependency-organization.md

[^bb-docs-intro]: Yocto Project. (n.d.). *BitBake User Manual — Introduction*. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-intro.html

[^bb-hello]: Yocto Project. (n.d.). *BitBake User Manual — Hello World Example*. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-hello.html

[^bb-guide]: a4z. (n.d.). *BitBake — a practical guide no Yocto*. Retrieved 2026-09-13, from https://a4z.noexcept.dev/docs/BitBake/guide.html

[^mailing]: OpenEmbedded bitbake-devel mailing list. (2023). *Using bitbake as a native package manager*. Retrieved 2026-09-13, from https://lists.openembedded.org/g/bitbake-devel/topic/using_bitbake_as_a_native/96539650