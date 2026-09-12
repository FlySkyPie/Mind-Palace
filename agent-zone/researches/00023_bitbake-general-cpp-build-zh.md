# BitBake 能否用於一般 C++ 建置而非嵌入式的研究

## 摘要

BitBake 是 Yocto Project 與 OpenEmbedded 專案共同維護的任務執行引擎與建置自動化工具，本質上為**建置完整嵌入式 Linux 發行版**而設計。雖然 BitBake 官方文件將其描述為「通用任務執行引擎」，技術上可在無 Yocto 環境下獨立運作，但對於一般 C++ 應用程式而言，其學習曲線極高、開銷龐大、社群支援稀少，且缺乏對 C++ 編譯原生的理解能力。結論是：**不建議將 BitBake 用作一般 C++ 專案的建置系統**，CMake、Meson 或 GNU Make 是更合適的選擇。

## 1. BitBake 是什麼及其主要用途

BitBake 是一個以 Python 撰寫的**任務執行引擎與建置自動化工具**，靈感來自 Gentoo Linux 的 Portage 套件管理系統。BitBake 最初是 OpenEmbedded 專案的一部分，於 2004 年拆分為獨立工具，目前由 Yocto Project 與 OpenEmbedded 共同維護[^wiki]。

官方 GitHub README 描述 BitBake 為：

> 「BitBake 是一個通用任務執行引擎，允許 shell 與 Python 任務在複雜的任務間相依約束下，以高效且並行的方式執行。」

然而，官方文件清楚指出其**主要用途**：

> 「執行 BitBake 的主要目的是產出某種輸出，例如單一可安裝套件、核心、軟體開發套件（SDK），甚至是完整的、特定於電路板的可開機 Linux 映像檔——包含開機載入程式、核心與根檔案系統。」[^yocto-intro]

BitBake 專為解決**建置嵌入式 Linux 發行版**所特有的問題而設計：
- **交叉編譯**處理
- **套件間相依性**（建置時期、原生時期與執行時期）
- 每個套件執行多個任務（擷取、解壓縮、修補、配置、編譯、安裝、打包）
- **架構無關**與**發行版無關**
- 支援**分層中繼資料**以實現模組化自訂

## 2. BitBake 能否用於一般 C++ 建置？

**技術上可以，但對大多數使用案例強烈不建議。**

BitBake 被描述為「通用任務執行引擎」，且已拆分為獨立、與發行版無關的工具，可在 Yocto/OpenEmbedded 環境外獨立使用。官方文件特別說明它被拆分為兩部分：「BitBake（通用任務執行器）」與「OpenEmbedded（BitBake 使用的中繼資料集）」[^yocto-intro]。

確實存在展示 BitBake 獨立使用（無 Yocto）的教學資源：

- Yocto Project 官方 BitBake 手冊包含一個 **Hello World 範例**（第 9 章），展示如何僅使用 BitBake 中繼資料來建置簡單的 C 程式，完全在 Yocto 環境之外[^hello]
- 一份**實務獨立 BitBake 指南**逐步引導建立從頭開始的完整 BitBake 專案（無 Yocto）[^guide]
- 有 YouTube 教學影片標題為「C++ Tutorial - Project build system using Bitbake (no YOCTOPROJECT)」
- 一個 GitHub 儲存庫 `adn-dodo/bitbake-tutorial-2026` 提供截至 2026 年最新的獨立 BitBake 教學[^tutorial]

然而，多個來源的共識相當明確：

該實務指南直接指出：

> 「理論上，由於 BitBake 執行程式碼，有人可能將 BitBake 用於建置軟體以外的事情，但這可能不是最好的主意。」[^guide]

Reddit 上 r/cpp 的討論中：

> 「僅僅因為可以執行一些 shell 指令就用 bitbake 來建置 C++ 專案並不是最好的主意，當你的專案有超過一個檔案時，問題就會開始出現。」[^reddit]

BitBake 開發郵件列表（bitbake-devel）上也有一個[公開討論串](https://lists.openembedded.org/g/bitbake-devel/topic/using_bitbake_as_a_native/96539650)，詢問 BitBake 是否可作為獨立的原生套件管理器（替代 Conan/vcpkg），顯示確實有人考慮過這點，但普遍回應是這並非該工具最佳化的用途[^mailing]。

## 3. BitBake 與其他建置系統的比較

### 3.1 設計哲學的根本差異

CMake、Meson 與 Make 是為**建置單一軟體專案**（或少數相互關聯的目標）而設計。BitBake 則是為**編排整個 Linux 發行版的建置**——數百個相互依賴的套件，各自擁有自己的建置系統，橫跨多個架構。

### 3.2 詳細比較

| 面向 | BitBake | CMake | Meson | GNU Make |
|------|---------|-------|-------|----------|
| **主要領域** | 嵌入式 Linux 發行版（多套件、交叉編譯） | 單專案建置（跨平台） | 單專案建置（現代 C/C++） | 通用建置工具 |
| **後設建置系統？** | 否（直接執行任務） | 是（產生 Makefile/Ninja） | 是（產生 Ninja 檔案） | 否（直接執行規則） |
| **語言** | BitBake DSL（Python + shell 在中繼資料中） | CMake DSL | Meson DSL（類似 Python） | Makefile 語法 |
| **學習曲線** | 非常陡峭 | 中等 | 中等 | 低（簡單專案） |
| **交叉編譯** | 一級支援，完全內建 | 透過 toolchain 檔案支援 | 支援 | 手動 |
| **相依性解析** | 完整 DAG 基礎，跨套件 | 依目標 | 依目標 | 基本 |
| **原始碼擷取** | 內建擷取器（git, http, svn 等） | 外部（FetchContent） | Wrap 系統 | 手動 |
| **平行化** | 完整平行任務執行 | 透過產生的 Ninja/Make | 透過 Ninja | 透過 `-j` 旗標 |
| **快取** | 基於簽章（共享狀態） | CCache 整合 | CCache 整合 | 手動 |
| **圖層系統** | 是（BBLAYERS, bbappend 檔案） | 否 | 否 | 否（子模組） |
| **適用於單一 C++ 應用程式** | ❌ 過度設計 | ✅ 極佳 | ✅ 極佳 | ✅ 良好 |
| **適用於多套件嵌入式 OS** | ✅ 為此設計 | ❌ 不適合 | ❌ 不適合 | ❌ 手動/腳本化 |

BitBake 官方手冊直接承認這個比較：

> 「概念上，BitBake 在某些方面類似 GNU Make，但有顯著差異：BitBake 根據提供的中繼資料來執行任務，這些中繼資料會建構出任務。」[^yocto-intro]

Yocto Wiki 也說明：

> 「BitBake 是一個建置工具——有點像 make，差別在於執行主要是在任務層級而非目標層級——每個目標可以有多個與之相關聯的任務，這些任務之間可以有相依性。」[^wiki]

## 4. 實際案例

1. **官方 BitBake Hello World**：Yocto Project 的 BitBake 手冊包含 Hello World 範例（第 9 章），使用 `do_compile` 與 `do_install` 任務建置簡單的 C 程式，完全在 Yocto 環境之外[^hello]。

2. **MultiTech 開發者資源**：其文件顯示一個「Hello World」配方直接使用 gcc 編譯 `helloworld.c` 檔案，並特別說明：「由於它不使用 autotools 或 make，我們必須明確告訴 BitBake 如何建置它。」[^multitech]

3. **郵件列表討論**：開發者詢問：「是否可能將 bitbake 用作獨立的套件管理器？作為 conan 或 vcpkg 等工具的替代品？想像一下，有一個軟體依賴於一些其他 C/C++ 函式庫，你想在主機系統上為此主機系統原生編譯它。」[^mailing]

4. **獨立 BitBake 指南**：`a4z.noexcept.dev` 的完整指南建立了一個獨立 BitBake 專案（無 Yocto），使用配方中的 shell 與 Python 任務來編譯軟體[^guide]。

## 5. 優缺點分析

### 5.1 可能考慮使用的優點

| 優點 | 說明 |
|------|------|
| **強大的相依性解析** | BitBake 建構完整的任務相依 DAG，自動解決套件間建置順序 |
| **平行執行** | BitBake 原生地獨立執行無關任務，最大化跨套件建置吞吐量 |
| **基於簽章的增量重建** | BitBake 的簽章系統（輸入的 checksum）可跳過未變更的任務，類似 Bazel 或 Pants |
| **圖層系統** | 可將建置邏輯模組化為圖層，搭配 `.bbappend` 檔案在不修改原始配方的情況下進行乾淨覆寫 |
| **內建擷取器** | BitBake 可自動從 git、http、svn 等來源擷取原始碼 |
| **交叉編譯支援** | 若需從桌面系統建置 ARM/RISC-V 目標，BitBake 原生處理 |
| **多配置建置** | 單次調用即可建置多個配置（除錯/釋出、不同架構） |

### 5.2 不建議使用的原因

| 缺點 | 說明 |
|------|------|
| **極陡的學習曲線** | BitBake 指南自承：「使用 BitBake 有相當陡峭的學習曲線。」需要理解 BBPATH、BBLAYERS、配方命名慣例、任務旗標、類別、附加檔案等——僅為了編譯一個 C++ 檔案[^guide] |
| **龐大的開銷** | 最小獨立 BitBake 專案需建立：`bblayers.conf`、`bitbake.conf`、`base.bbclass`、`layer.conf` 加上配方。官方 Hello World 範例需要 12 個步驟僅為印出文字[^hello] |
| **僅限 Linux** | BitBake 僅在 Linux 上運作。不支援 Windows 或 macOS（與 CMake/Meson 不同） |
| **Python 相依性** | 需要 Python 3.9+ 並正確設定 PATH/PYTHONPATH |
| **無原生 C/C++ 建置智慧** | BitBake 不原生理解 C/C++ 編譯慣用語。你必須在 BitBake 配方中撰寫 shell 指令來處理編譯、連結、include 路徑等——本質上是在 BitBake 內部撰寫 Make/CMake 邏輯 |
| **不利於專案管理** | BitBake 為**發行版**建置而設計，非**專案**建置。用 BitBake 管理單一 C++ 儲存庫的原始碼檔案，遠不如 CMake 的 `add_executable()` 或 Meson 的 `executable()` 來得直觀 |
| **獨立使用的文件不足** | 多數 BitBake 文件假設 Yocto/OpenEmbedded 情境，尋找獨立使用範例需耗費大量心力 |
| **此用途的社群極小** | 幾乎沒有人將 BitBake 獨立用於 C++ 專案，難以找到幫助、範例或函式庫生態系 |
| **版本相容性** | BitBake 中繼資料「通常向後相容但非向前相容」——使用錯誤的 BitBake 版本搭配中繼資料將會失敗 |
| **快取/建置目錄污染** | BitBake 會建立龐大的 `tmp/` 目錄，包含所有任務日誌、戳記與工作目錄，遠比一般的 `build/` 資料夾開銷更大 |

## 6. 結論

**BitBake 不建議作為一般 C++ 應用程式的通用建置系統。**

嘗試過此做法的少數人將其描述為不是好主意[^reddit]。BitBake 是一個專門工具，專為編排包含數百個相互依賴套件、具備交叉編譯需求的**完整嵌入式 Linux 發行版**建置而設計。

對於典型的 C++ 應用程式或函式庫專案：
- **CMake** 是事實標準（跨平台、龐大生態系、IDE 支援）[^so]
- **Meson** 是強勁的現代替代方案（更快速、更簡潔的語法、適合中大型專案）
- **GNU Make** 適合較簡單的專案
- **Bazel** 或 **Pants** 若需要類似 BitBake 的先進相依快取與平行建置功能，會是更好的選擇

**如果你已經深陷 Yocto 生態系**且需要在嵌入式 Linux 系統中建置 C++ 元件，那麼使用 BitBake 配方是正確的做法——但那正是 BitBake 的設計目的。但對於獨立 C++ 開發，請選擇為此目的設計的建置系統。

## 參考資料

[^wiki]: Wikipedia. (n.d.). *BitBake*. Retrieved 2026-09-13, from https://en.wikipedia.org/wiki/BitBake

[^yocto-intro]: Yocto Project. (n.d.). *BitBake User Manual — Introduction*. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-intro.html

[^hello]: Yocto Project. (n.d.). *BitBake User Manual — Hello World Example*. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-hello.html

[^guide]: a4z. (n.d.). *BitBake — a practical guide no Yocto*. Retrieved 2026-09-13, from https://a4z.noexcept.dev/docs/BitBake/guide.html

[^tutorial]: adn-dodo. (2026). *bitbake-tutorial-2026*. Retrieved 2026-09-13, from https://github.com/adn-dodo/bitbake-tutorial-2026

[^reddit]: Reddit r/cpp. (2021). *Simple build system using bitbake for beginners*. Retrieved 2026-09-13, from https://www.reddit.com/r/cpp/comments/kqw0h7/simple_build_system_using_bitbake_for_beginners/

[^mailing]: OpenEmbedded bitbake-devel mailing list. (2023). *Using bitbake as a native package manager*. Retrieved 2026-09-13, from https://lists.openembedded.org/g/bitbake-devel/topic/using_bitbake_as_a_native/96539650

[^multitech]: MultiTech. (n.d.). *Writing BitBake Recipes*. Retrieved 2026-09-13, from https://www.multitech.net/developer/software/corecdp/development/writing-bitbake-recipes/

[^so]: Stack Overflow. (2013). *BitBake vs CMake for x86 and ARM project*. Retrieved 2026-09-13, from https://stackoverflow.com/questions/19304224/bitbake-vs-cmake-for-x86-and-arm-project