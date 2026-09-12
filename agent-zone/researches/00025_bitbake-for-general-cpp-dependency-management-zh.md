# BitBake 用於一般 C++ 依賴管理的可行性研究

BitBake 是 Yocto Project 和 OpenEmbedded 底層的工作執行引擎，理論上可作為通用任務調度器，但將其用於一般（非嵌入式）C++ 專案的依賴與建置管理，**在實務上非常不合適**。

## BitBake 的本質與定位

BitBake 是一個基於 Python 的**通用任務執行引擎**，能高效並行執行 shell 與 Python 任務，並處理複雜的任務間依賴關係[^bitbake-intro]。其設計靈感來自 Gentoo Linux 的 Portage 套件管理系統[^wikipedia]。

BitBake 的核心功能包括：
- **配方（Recipe, `.bb`）**：描述如何抓取、修補、配置、編譯、安裝與打包軟體
- **類別（Class, `.bbclass`）**：共享功能模組（如 `base.bbclass` 提供 `do_fetch`, `do_unpack`, `do_configure`, `do_compile`, `do_install` 等預設任務）
- **圖層（Layer）**：模組化、可覆蓋的配方與配置集合
- **附加檔（`.bbappend`）**：在不修改原始配方的情況下擴展/覆蓋
- **內建擷取器**：支援 git、http、ftp、CVS、SVN、本地檔案等
- **簽章快取（sstate）**：透過輸入的校驗和加速重建
- **一級交叉編譯支援**：可為主機完全不同的架構建置

官方文件明確指出 BitBake「試圖在系統層面上保持儘可能獨立」[^bitbake-intro]，而 README 也稱其為「通用任務執行引擎」[^bitbake-readme]。

## 能否用於一般 C++ 依賴管理？

**技術上可行，但實務上是糟糕的選擇。**

### 為什麼技術上可行

由於 BitBake 只是任務執行引擎，你可以：
- 編寫配方下載 C++ 函式庫原始碼並編譯
- 透過 `DEPENDS` 定義配方間的依賴鏈
- 使用內建擷取器從 git/HTTP 拉取
- 利用並行任務執行引擎加速

如同某份實用指南所言：「理論上，由於 BitBake 執行程式碼，有人可以用 BitBake 做軟體建置以外的事情，但這很可能不是最好的主意。」[^practical-guide]

### 為什麼實務上不適合

- **無 C++ 意識**：BitBake 不理解 C++ 編譯單元、標頭檔、包含路徑或函式庫。你必須手動呼叫編譯器，缺乏 CMake 或 Meson 等建置系統的直接整合[^roundup]。
- **無套件註冊表**：不同於 Conan 或 vcpkg，BitBake 沒有集中式的預編譯 C++ 套件註冊表。所有套件都從原始碼建置。
- **極高設定開銷**：即使是最簡單的「hello world」，也需要建立圖層、`bblayers.conf`、`bitbake.conf`、`base.bbclass`、`layer.conf` 和配方，還需要執行 Python 服務程序[^practical-guide]。
- **陡峭學習曲線**：BitBake 的 DSL（變數、覆蓋語法、任務依賴、圖層系統）複雜且專屬於嵌入式 Linux 領域。
- **Linux 主機為主**：BitBake 主要針對 Linux 設計，Windows/macOS 支援非一級體驗。
- **無 IDE 整合**：沒有 VS、CLion、Qt Creator、Xcode 等 IDE 的整合。
- **冷啟動慢**：解析所有配方和中繼資料即使在小型專案也需要相當時間。
- **文件假設 Yocto/OpenEmbedded 背景**：所有範例與教學都假設在建置嵌入式 Linux 的脈絡下。

## 優缺點總結

### 優點

- 強大的依賴解析（建置期、執行期、原生與目標依賴分別處理）
- 內建多種擷取器（git、http、ftp、SVN、CVS、本地檔案）
- 高效的並行任務執行
- 簽章式快取（sstate），避免未變更輸入的重建
- 嚴格的再現性（所有輸入都有校驗和）
- 圖層化架構，乾淨的關注點分離與覆蓋機制
- 支援多目標/多架構建置

### 缺點

- 極高的設定開銷與專案結構複雜度
- 無 C++ 生態系整合（不原生支援 CMake、Conan、vcpkg、Meson）
- 整個工具圍繞交叉編譯 Linux 發行版設計
- 陡峭的學習曲線與晦澀的 DSL 語法
- 預設從原始碼建置，無二進位套件分發機制
- 非跨平台桌面友好（主要 Linux）
- 冷啟動解析速度慢
- 所有文件與範例假設嵌入式的 Yocto 背景
- 無 IDE 整合

## 與主流 C++ 依賴管理工具的比較

| 能力 | BitBake | vcpkg | Conan | CMake FetchContent |
|------|---------|-------|-------|-------------------|
| **主要用途** | 建置嵌入式 Linux 發行版 | C/C++ 套件管理 | C/C++ 套件管理 | CMake 原生原始碼抓取 |
| **C++ 意識** | 無（shell 任務） | 深（CMake 整合） | 深（所有建置系統產生器） | 完整（CMake targets） |
| **二進位快取** | 有（sstate） | 有（binary cache） | 有（Artifactory） | 無（從原始碼建置） |
| **套件註冊表** | 無獨立註冊表 | 大（2200+ ports） | 大（ConanCenter） | 無 |
| **IDE 整合** | 無 | VS, VS Code, CLion | VS Code, CLion | 所有 CMake 相容 IDE |
| **跨平台** | 主要 Linux | Windows, Linux, macOS | Windows, Linux, macOS | 所有 |
| **學習曲線** | 非常陡峭 | 低 | 中 | 低 |
| **設定開銷** | 非常高 | 低（`vcpkg install`） | 低（`conan install`） | 極低（CMake 函式） |
| **依賴解析** | SAT 類（任務式） | Port 式 | SAT 求解器 | 無（手動） |
| **版本支援** | 配方檔案名稱版本 | Port 版本 | 完整 semver | Git tag/commit |

## 結論

BitBake 是 Yocto/OpenEmbedded 生態系中的強大工具，但其設計目標與一般 C++ 桌面/伺服器應用開發截然不同。對於常規 C++ 專案：

- **建置系統**：使用 **CMake**（事實標準）或 **Meson**（現代替代方案）
- **套件管理**：使用 **Conan**（最靈活、企業級）或 **vcpkg**（最簡單，尤其在 Windows）
- **CMake 原生依賴抓取**：使用 **FetchContent** 或 **CPM.cmake**（輕量、無外部工具）
- **Meson 用戶**：使用 **Meson wraps**（WrapDB）

BitBake 的唯一適用場景是**建置嵌入式 Linux 發行版**。脫離此場景，它帶來的複雜度遠超過其價值。

[^bitbake-intro]: Yocto Project. (n.d.). BitBake User Manual. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-intro.html

[^wikipedia]: Wikipedia. (n.d.). BitBake. Retrieved 2026-09-13, from https://en.wikipedia.org/wiki/BitBake

[^bitbake-readme]: OpenEmbedded. (n.d.). BitBake README. Retrieved 2026-09-13, from https://github.com/openembedded/bitbake

[^practical-guide]: A Practical Guide to BitBake. (n.d.). Retrieved 2026-09-13, from https://a4z.noexcept.dev/docs/BitBake/guide.html

[^roundup]: Modern C++ DevOps. (n.d.). C++ Package Managers Roundup. Retrieved 2026-09-13, from https://moderncppdevops.com/pkg-mngr-roundup/