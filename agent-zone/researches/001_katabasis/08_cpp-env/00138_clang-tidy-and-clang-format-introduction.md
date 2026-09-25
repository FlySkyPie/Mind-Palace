# Clang-Tidy 與 Clang-Format 介紹

## 概述

Clang-Tidy 與 Clang-Format 是 LLVM/Clang 專案中的兩款 C/C++ 程式碼品質工具，分別處理**語意分析**與**格式排版**兩個互補的面向。前者是靜態分析器（linter），負責揪出程式碼中的錯誤、壞味道、效能問題；後者是自動格式化工具，負責統一縮排、括號、換行等排版風格。兩者常搭配使用，形成程式碼品質的自動化閘門（quality gate）。

---

## Clang-Format：自動化程式碼排版

### 用途與原理

Clang-Format 是一款基於 Clang 前端（LibFormat）的程式碼格式化工具，支援 C、C++、Java、JavaScript、Objective-C、Protobuf、C# 等多種語言[^clang-format-docs]。它將原始碼解析為語法樹後，根據設定的規則重新輸出排版後的文字——包括縮排、換行、空格、括號位置、指標置左/置右等。它**不改變程式碼的語意**，只改變外觀。

### 常用功能

- **就地修改**：`clang-format -i file.cpp` 直接改寫檔案
- **僅檢查**：`clang-format -n file.cpp` （或 `--dry-run`）只顯示差異不修改
- **Git 整合**：`git-clang-format` 僅格式化 Git 有變更的行；`clang-format-diff.py` 解析 unified diff 只改動差異部分
- **選擇性停用**：以 `// clang-format off` 和 `// clang-format on` 包住不需要格式化的區塊[^clang-format-docs]

### 設定檔 `.clang-format`

Clang-Format 使用 YAML 格式的 `.clang-format` 檔案（Windows 可用 `_clang-format`）。工具會從原始碼目錄往上層目錄搜尋此檔案[^clang-format-style]。

內建的預設風格（preset）包括[^clang-format-style]：

| 風格 | 特色 |
|---|---|
| LLVM | 2 空格縮排、80 字元限制、指標靠右 |
| Google | 2 空格縮排、80 字元限制、指標靠右，符合 Google C++ Style Guide |
| Chromium | 近似 Google 但有 Chromium 專屬調整 |
| Mozilla | 不同的括號與縮排規則 |
| WebKit | 獨特的括號與空格規則 |
| Microsoft | 符合 Microsoft 慣例 |
| GNU | 較多縮排，符合 GNU Coding Standards |

使用方式：
```bash
# 從 Google 風格產生 .clang-format
clang-format -style=google -dump-config > .clang-format

# 直接套用 Google 風格
clang-format -style=google -i *.cpp

# 混用自訂選項
clang-format -style="{BasedOnStyle: google, IndentWidth: 4}" -i *.cpp
```

`.clang-format` 還有一些常用選項：`IndentWidth`（縮排寬度）、`ColumnLimit`（欄位上限，0 為不限）、`UseTab`（是否使用 Tab）、`PointerAlignment`（指標 `*` 靠左/靠右/置中）、`SortIncludes`（自動排序 `#include`）、`BreakBeforeBraces`（括號換行策略）[^clang-format-style]。

---

## Clang-Tidy：語意級靜態分析

### 用途與原理

Clang-Tidy 是一個基於 Clang AST（抽象語法樹）的 lint 工具與靜態分析器[^clang-tidy-docs]。與僅處理排版的 Clang-Format 不同，Clang-Tidy 能理解程式碼的**語意**——型別、變數壽命、移動語意、記憶體操作等——並對數百種常見錯誤與壞味道發出警告，許多檢查還能自動修復（`--fix`）。

由於需要理解型別與巨集，Clang-Tidy 需要**編譯資料庫**（`compile_commands.json`），可由 CMake 以 `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` 產生[^clang-tidy-docs]。

### 檢查類別

Clang-Tidy 的檢查（check）按前綴分為數十個類別[^clang-tidy-checks]：

| 前綴 | 說明 |
|---|---|
| `bugprone-*` | 容易出錯的程式碼模式（如 use-after-move、窄化轉型、參數順序錯誤） |
| `modernize-*` | 建議使用現代 C++ 功能（如 `override`、`auto`、`make_unique`、range-based for） |
| `performance-*` | 效能問題（如不必要的複製、應傳 const-ref 卻傳值、vector 低效增長） |
| `readability-*` | 可讀性（如命名慣例、應使用 `.empty()` 而非 `.size()>0`、函式過長） |
| `clang-analyzer-*` | Clang Static Analyzer 的深度分析（如 null 解參考、記憶體洩漏、buffer overflow） |
| `cert-*` | CERT 安全編碼標準 |
| `cppcoreguidelines-*` | C++ Core Guidelines |
| `concurrency-*` | 並行程式問題 |
| `misc-*` | 無法歸入上述類別的其他檢查 |
| `google-*` / `llvm-*` | 各專案的編碼慣例 |

### 使用方式

```bash
# 搭配 compile_commands.json
clang-tidy source.cpp -p=build/

# 指定檢查項目（-* 關閉預設，再啟用特定類別）
clang-tidy source.cpp -checks="-*,bugprone-*,modernize-*,performance-*"

# 自動修復
clang-tidy source.cpp -checks=modernize-* --fix

# 差異分析（CI 友善）
git diff -U0 HEAD^ | clang-tidy-diff.py -p1

# 並行分析整個專案
run-clang-tidy.py -p=build/ -j 4
```

### 設定檔 `.clang-tidy`

也是 YAML 格式，從原始碼目錄往上層搜尋，預設會繼承上層目錄的設定（`InheritParentConfig: true`）[^clang-tidy-docs]。

```yaml
---
Checks: '-*,bugprone-*,modernize-*,performance-*,readability-*'
WarningsAsErrors: '*'
HeaderFilterRegex: '.*'
FormatStyle: file
CheckOptions:
  readability-identifier-naming.ClassCase: CamelCase
  readability-identifier-naming.FunctionCase: camelBack
  readability-function-size.LineThreshold: 80
```

### 行內抑制

```cpp
int x; // NOLINT
int y; // NOLINT(google-explicit-constructor)
// NOLINTNEXTLINE(bugprone-use-after-move)
auto val = std::move(x);
// NOLINTBEGIN(readability-function-size)
void big() { /* ... */ }
// NOLINTEND(readability-function-size)
```

---

## 兩者比較

| 面向 | Clang-Format | Clang-Tidy |
|---|---|---|
| 主要角色 | 程式碼排版（縮排、空格、換行） | Linter／靜態分析（偵測錯誤、壞味道） |
| 處理層級 | 語法樹的**布局** | 語法樹的**語意** |
| 解決的問題 | 縮排不一致、括號風格、行長 | 未初始化變數、記憶體洩漏、缺少 `override`、效能問題 |
| 自動修復 | `-i` 就地改寫 | `--fix` 套用修復 |
| 需 compile_commands.json | 不需要 | 需要 |
| 設定檔 | `.clang-format` | `.clang-tidy` |
| 速度 | 極快（純語法層級） | 較慢（需完整語意分析） |

## 兩者如何互補

Clang-Format 與 Clang-Tidy 處理程式碼品質的**不同維度**——前者管外觀，後者管正確性——因此常串接在同一條 pipeline 中使用[^labri-comparison]：

```bash
# 典型的完整流程
clang-format -i --style=file source.cpp         # 1. 格式化
clang-tidy source.cpp --fix -- -std=c++20       # 2. 語意分析與修復
clang-format -i --style=file source.cpp         # 3. 修復後重新格式化
```

或者讓 Clang-Tidy 直接用 Clang-Format 的風格來格式化它所插入的修復：

```bash
clang-tidy source.cpp --fix --format-style=file -- -std=c++20
```

兩者搭配的具體效益：

1. **消除程式碼審查中的噪音**：審查者不再需要花力氣看縮排、括號位置或忘記加的 `override`，可以專注在架構與演算法[^labri-comparison]。
2. **涵蓋對方的盲區**：Clang-Format 無法檢查命名慣例或偵測 bug；Clang-Tidy 無法強制縮排一致性。
3. **共享設定生態系**：兩者的 YAML 設定檔都放在專案根目錄並納入版本控制，形成團隊共識的自動化強制力。
4. **CI 整合**：`git-clang-format` 與 `clang-tidy-diff.py` 可作為 pre-commit hook 或 CI 檢查，只在變更的行上執行，速度夠快不影響開發流程。

## 總結

- **Clang-Format**：自動化排版，解決程式碼外觀一致性問題，基於語法樹布局，不需要編譯資料庫，執行極快。
- **Clang-Tidy**：語意級靜態分析，偵測 bug、壞味道、效能問題、現代化機會，需要編譯資料庫，提供自動修復。
- **兩者互補**：在開發流程中串聯使用，可同時確保程式碼的**形式統一**與**內容正確**，讓審查者將精力放在真正重要的設計問題上。

---

[^clang-format-docs]: clang.llvm.org. (n.d.). ClangFormat. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormat.html
[^clang-format-style]: clang.llvm.org. (n.d.). Clang-Format Style Options. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormatStyleOptions.html
[^clang-tidy-docs]: clang.llvm.org. (n.d.). Clang-Tidy — Extra Clang Tools. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/
[^clang-tidy-checks]: clang.llvm.org. (n.d.). Clang-Tidy Checks — Full List. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/checks/list.html
[^labri-comparison]: Labri.fr. (n.d.). Using Clang-Tidy and Clang-Format. Retrieved 2026-09-25, from https://www.labri.fr/perso/fleury/posts/programming/using-clang-tidy-and-clang-format.html