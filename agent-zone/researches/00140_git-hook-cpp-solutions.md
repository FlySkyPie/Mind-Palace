# Git Hook 工具選型研究：C++ 專案適用方案

## 概述

本文研究適用於 C++ 專案的 Git Hook 管理工具，比較主流的 **pre-commit**、**Husky** 與 **Lefthook** 三套方案在 C++ 生態圈的支援程度與使用體驗。

## 工具比較總覽

### 1. pre-commit

Git Hook 生態圈中最受歡迎的框架，透過 `.pre-commit-config.yaml` 設定，自動管理 hook 環境與依賴。在 C++ 專案中有豐富的預建 hook 可用，且社群活躍，是目前 C++ 專案最推薦的方案。[^precommit][^precommit-hooks]

**C++ 相關 Hook 生態系（最完善）**：

| Hook | 來源 | 功能 |
|---|---|---|
| **clang-format** | `pre-commit/mirrors-clang-format` | C/C++/CUDA 格式化，官方維護的 pip wheel 版本 |
| **clang-tidy** | `pocc/pre-commit-hooks` | 靜態分析，支援 compilation database |
| **cppcheck** | `pocc/pre-commit-hooks` | 靜態分析（未使用變數等） |
| **cpplint** | `pocc/pre-commit-hooks` | Google 風格的 C++ 程式碼風格檢查 |
| **include-what-you-use** | `pocc/pre-commit-hooks` | `#include` 最佳化 |
| **cmake-format** | `cheshirekow/cmake-format-precommit` | CMakeLists.txt 格式化 |
| **cmake-lint** | `cheshirekow/cmake-format-precommit` | CMakeLists.txt 語法檢查 |
| **Compilation DB 自動產生** | `Takishima/cmake-pre-commit-hooks` | 自動產生 `compile_commands.json` 後執行 clang-tidy/cppcheck/iwyu |

**優點**：
- 預建 hook 生態系最豐富，C++ 工具可直接使用
- 自動管理各 hook 的依賴環境
- 支援版本鎖定與自動更新 (`pre-commit autoupdate`)
- 良好的 CI 整合（`pre-commit.ci`、GitHub Actions）

**缺點**：
- 框架本身需要 Python 環境
- 預設序列執行（較慢）
- 部分 hook 仍需本機安裝對應工具（如 clang-tidy、cppcheck）

**C++ 適用性：★★★★★（極佳）**

---

### 2. Husky

現代化輕量 Git Hook 管理器，使用 `core.hooksPath` 功能將 hook 腳本指向 `.husky/` 目錄。[^husky][^husky-getstarted]

Hook 即為純 POSIX Shell 腳本，無預建 hook 生態。安裝需透過 npm，主要設計給 Node.js 專案使用。

**C++ 適用性：★★☆☆☆（不適合）**

- 無預建 C++ hook，所有 hook 需手動以 shell 腳本撰寫
- clang-tidy 需 compilation database，無自動產生支援
- 無並行執行能力
- 需 Node.js/npm 安裝

---

### 3. Lefthook

Go 語言撰寫的單一二進位檔案 Git Hook 管理器，無執行期依賴。透過 `lefthook.yml`/`lefthook.toml`/`lefthook.json` 設定。[^lefthook][^lefthook-config]

**優點**：
- 並行執行（多個 hook 同時跑，clang-format + cppcheck + cmake-format 可同時進行）
- 單一二進位檔，無執行期依賴
- 支援 Glob/Regex 檔案過濾、Docker runner、本地設定覆蓋
- `{staged_files}`、`{all_files}`、`{push_files}` 樣板變數靈活

**缺點**：
- 無預建 hook 生態，所有 hook 需手動設定
- 工具（clang-format、clang-tidy）需在本機 PATH 上

**C++ 適用性：★★★★☆（良好）**

---

### 4. C++ 專用 Git Hook 方案

除了上述通用框架，以下專案也值得關注：

- **`pocc/pre-commit-hooks`**[^pocc]：最完整的 C/C++ pre-commit hook 集合，403 stars。正確處理 exit code、`--version` 強制、cppcheck 持久快取、compilation database 支援。
- **`Takishima/cmake-pre-commit-hooks`**[^takishima]：CMake 感知 hook，自動產生 compilation database 再執行分析工具。支援 TOML 設定、`--all-at-once` 模式、跨平台 CMake 選項。也提供 `pip install cmake-pre-commit-hooks` CLI 工具。
- **`cheshirekow/cmake-format-precommit`**[^cheshirekow]：`cmake-format`/`cmake-lint` 的官方 pre-commit hook。

## 綜合對比

| 特性 | pre-commit | Husky | Lefthook |
|---|---|---|---|
| 底層語言 | Python | Shell 腳本 | Go（二進位） |
| 預建 C++ hook | ✅ 豐富 | ❌ 無 | ⚠️ 無，易自訂 |
| clang-format | ✅ 官方 mirror | ❌ 手動 | ⚠️ 手動設定 |
| clang-tidy | ✅ 經 pocc | ❌ 手動 | ⚠️ 手動設定 |
| cmake-format | ✅ 官方 | ❌ 手動 | ⚠️ 手動設定 |
| Compilation DB | ✅ 經 Takishima | ❌ | ❌ 需手動 |
| 並行執行 | ❌ 序列 | ❌ | ✅ 內建 |
| 檔案過濾 | ✅ types/glob | ❌ 手動 | ✅ Glob/Regex |
| 環境管理 | ✅ 自動安裝 | ❌ | ❌ |
| C++ 設定成本 | ✅ 加入設定即可 | ❌ 需撰寫腳本 | ⚠️ 需撰寫設定 |
| 開銷 | ~100ms+ | ~1ms | ~1ms |
| GitHub Stars | ~20k+ | ~35k | ~9k |
| 授權 | MIT | MIT | MIT |

## 結論與建議

對於 C++ 專案，**pre-commit 是最推薦的方案**，原因如下：

1. **預建 hook 生態系最豐富**：clang-format、clang-tidy、cppcheck、cmake-format 等工具皆有官方或社群維護的現成 hook，開箱即用。
2. **環境自動管理**：每個 hook 的依賴會自動安裝與隔離，減少開發者本機環境設定不一致的問題。
3. **compilation database 支援**：透過 `Takishima/cmake-pre-commit-hooks` 可自動產生 C++ 工具所需的 `compile_commands.json`。
4. **社群與成熟度**：超過 20k stars，廣泛的 CI 整合支援，長期維護。

**Lefthook 是良好替代方案**，若團隊偏好以下特性可考慮：
- 不需要 Python 依賴，希望單一二進位檔即可使用
- 需要並行執行 hook 以提升速度
- 團隊願意手動撰寫設定檔

**Husky 不推薦用於 C++ 專案**，除非該專案本身已是 Node.js/npm 為主的環境，僅需簡單的 C++ 格式化檢查。

## 參考資料

[^precommit]: pre-commit. (n.d.). pre-commit. Retrieved 2026-09-25, from https://pre-commit.com
[^precommit-hooks]: pre-commit. (n.d.). Supported hooks. Retrieved 2026-09-25, from https://pre-commit.com/hooks.html
[^pocc]: pocc. (n.d.). pocc/pre-commit-hooks. Retrieved 2026-09-25, from https://github.com/pocc/pre-commit-hooks
[^takishima]: Takishima. (n.d.). Takishima/cmake-pre-commit-hooks. Retrieved 2026-09-25, from https://github.com/Takishima/cmake-pre-commit-hooks
[^cheshirekow]: cheshirekow. (n.d.). cheshirekow/cmake-format-precommit. Retrieved 2026-09-25, from https://github.com/cheshirekow/cmake-format-precommit
[^husky]: typicode. (n.d.). Husky. Retrieved 2026-09-25, from https://typicode.github.io/husky
[^husky-getstarted]: typicode. (n.d.). Husky get started. Retrieved 2026-09-25, from https://typicode.github.io/husky/get-started.html
[^lefthook]: evilmartians. (n.d.). Lefthook. Retrieved 2026-09-25, from https://lefthook.dev
[^lefthook-config]: evilmartians. (n.d.). Lefthook configuration. Retrieved 2026-09-25, from https://lefthook.dev/configuration/
[^clang-format-wheel]: ssciwr. (n.d.). clang-format-wheel. Retrieved 2026-09-25, from https://github.com/ssciwr/clang-format-wheel
[^cmake-format-docs]: cmakelang. (n.d.). cmake-format installation. Retrieved 2026-09-25, from https://cmake-format.readthedocs.io/en/latest/installation.html