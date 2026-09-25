# C++ 專案整合 clang-tidy 與 clang-format 程式碼風格檢查工具指南

## 概述

在 C++ 專案中維持一致的程式碼風格與靜態分析至關重要。clang-format 負責自動格式化程式碼（縮排、空格、換行等純機械性變更），而 clang-tidy 則提供超過 300 種靜態分析檢查（包含潛在錯誤、效能問題、現代 C++ 用法等），兩者皆為 LLVM 專案的一部分。[^clang-format-official][^clang-tidy-official]

## 1. 設定檔建立

### `.clang-format` 設定檔

可從預設風格產生完整設定檔，再依需求調整：

```bash
clang-format -style=llvm -dump-config > .clang-format
```

常用預設風格包含：`LLVM`、`Google`、`Chromium`、`Mozilla`、`WebKit`、`Microsoft`、`GNU`。[^clang-format-style]

實例（以 Google 為基礎微調）：

```yaml
# .clang-format
BasedOnStyle: Google
IndentWidth: 4
ColumnLimit: 100
AccessModifierOffset: -4
AllowShortFunctionsOnASingleLine: Inline
SortIncludes: true
PointerAlignment: Left
UseTab: Never
FixNamespaceComments: true
```

亦可使用 `.clang-format-ignore` 檔案排除特定檔案（支援 glob 模式）。[^clang-format-ignore]

### `.clang-tidy` 設定檔

```yaml
# .clang-tidy
Checks: '-*,
  bugprone-*,
  performance-*,
  modernize-*,
  readability-*,
  cppcoreguidelines-*,
  clang-analyzer-*'
WarningsAsErrors: '*'
HeaderFilterRegex: '.*'
FormatStyle: none
```

`-*` 表示先停用所有檢查，再逐行啟用需要的群組，可避免規則衝突。[^clang-tidy-config]

## 2. CMake 整合

### clang-tidy 整合（CMAKE_CXX_CLANG_TIDY）

CMake 3.6+ 支援直接透過變數讓 clang-tidy 在編譯時自動執行：

```cmake
# CMakeLists.txt - 最簡整合
set(CMAKE_CXX_CLANG_TIDY "clang-tidy")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

亦可指定檢查項目與將警告視為錯誤：

```cmake
find_program(CLANG_TIDY_EXE "clang-tidy")
if(CLANG_TIDY_EXE)
  set(CMAKE_CXX_CLANG_TIDY ${CLANG_TIDY_EXE}
    -checks=-*,bugprone-*,performance-*,modernize-*
    -warnings-as-errors=*)
endif()
```

CMake 3.27+ 支援 Generator Expression 與逐檔案跳過檢查：

```cmake
# 只在 Debug 模式啟用
set(CMAKE_CXX_CLANG_TIDY
    "clang-tidy"
    $<$<CONFIG:Debug>:-warnings-as-errors=*>)

# 跳過特定檔案
set_source_files_properties(generated.cpp PROPERTIES SKIP_LINTING TRUE)
```

CMake 3.24+ 支援匯出修正檔：

```cmake
set(CMAKE_CXX_CLANG_TIDY_EXPORT_FIXES_DIR ${CMAKE_BINARY_DIR}/tidy-fixes)
```

[^cmake-clang-tidy][^cmake-lang-clang-tidy]

### clang-format 整合（自訂 CMake Target）

```cmake
find_program(CLANG_FORMAT_EXE clang-format)
if(CLANG_FORMAT_EXE)
  file(GLOB_RECURSE ALL_SOURCE_FILES src/*.cpp include/*.hpp)

  add_custom_target(format
    COMMAND ${CLANG_FORMAT_EXE} -i -style=file ${ALL_SOURCE_FILES}
    COMMENT "Formatting with clang-format")

  add_custom_target(format-check
    COMMAND ${CLANG_FORMAT_EXE} --dry-run --Werror -style=file ${ALL_SOURCE_FILES}
    COMMENT "Checking formatting compliance")
endif()
```

[^ttroy50-cmake-examples]

## 3. Pre-Commit Hook 整合

### 推薦：使用 pre-commit 框架 + cpp-linter-hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/cpp-linter/cpp-linter-hooks
    rev: v1.6.0
    hooks:
      - id: clang-format
        args: [--style=file, --version=21]
        files: ^(src|include)/.*\.(cpp|cc|cxx|h|hpp)$
      - id: clang-tidy
        args: [--checks=.clang-tidy, --version=21]
        files: ^(src|include)/.*\.(cpp|cc|cxx|h|hpp)$
```

`cpp-linter-hooks` 會自動安裝 Clang 工具（無需手動安裝 LLVM），並自動偵測 `build/compile_commands.json`。[^cpp-linter-hooks]

若僅需 clang-format，也可使用較簡潔的 mirror：

```yaml
- repo: https://github.com/pre-commit/mirrors-clang-format
  rev: v21.1.8
  hooks:
    - id: clang-format
      args: [--style=file]
```

[^pre-commit-clang-format]

## 4. CI/CD 整合（GitHub Actions）

### 推薦：cpp-linter-action（功能最完整）

```yaml
# .github/workflows/cpp-linter.yml
name: C++ Lint & Format
on: [push, pull_request]

jobs:
  cpp-linter:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      checks: write
    steps:
      - uses: actions/checkout@v5
      - name: Configure CMake
        run: cmake -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
      - uses: cpp-linter/cpp-linter-action@v2
        id: linter
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          style: 'file'            # 使用 .clang-format
          tidy-checks: ''          # 使用 .clang-tidy
          version: '21'
          thread-comments: ${{ github.event_name == 'pull_request' && 'update' }}
          step-summary: true
      - name: Fail on errors
        if: steps.linter.outputs.checks-failed > 0
        run: exit 1
```

此 Action 支援 PR 行內註解、自動修正、步驟摘要等功能。[^cpp-linter-action]

### 簡潔版：手動 Workflow

```yaml
name: Format Check
on: [push, pull_request]
jobs:
  formatting-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: jidicula/clang-format-action@v4.18.0
        with:
          clang-format-version: '18'
          check-path: 'src'
          fallback-style: 'LLVM'
```

[^clang-format-action]

## 5. Makefile 整合

```makefile
CLANG_FORMAT ?= clang-format
CLANG_TIDY   ?= clang-tidy
SOURCES      := $(shell find src/ -name '*.cpp' -o -name '*.hpp')
BUILD_DIR    := build

.PHONY: format format-check tidy all

format:
	$(CLANG_FORMAT) -i -style=file $(SOURCES)

format-check:
	$(CLANG_FORMAT) --dry-run --Werror --style=file $(SOURCES)

tidy: $(BUILD_DIR)/compile_commands.json
	$(CLANG_TIDY) $(SOURCES) -p $(BUILD_DIR) --warnings-as-errors=*

$(BUILD_DIR)/compile_commands.json:
	cmake -S . -B $(BUILD_DIR) -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

## 6. 命令列手動執行

### clang-format

```bash
# 格式化單一檔案（in-place）
clang-format -i --style=file source.cpp

# 乾執行（僅報告不符格式的檔案）
clang-format --dry-run --Werror --style=file source.cpp

# 格式化所有原始檔
find src/ -name '*.cpp' -o -name '*.hpp' | xargs clang-format -i --style=file

# 僅格式化 Git 更動的行
git clang-format
```

[^clang-format-commandline]

### clang-tidy

```bash
# 對單一檔案執行
clang-tidy source.cpp -p build/

# 自動套用修正
clang-tidy source.cpp -p build/ --fix

# 多執行緒對整個專案執行
run-clang-tidy.py -p build/ -j $(nproc)

# 僅檢查 Git diff 中的行
git diff -U0 HEAD | clang-tidy-diff.py -p1

# 驗證設定檔
clang-tidy --verify-config
```

[^clang-tidy-commandline]

### NOLINT 抑制註解

```cpp
// 抑制同一行所有檢查
int x; // NOLINT

// 抑制特定檢查
int y; // NOLINT(readability-magic-numbers)

// 抑制下一行
// NOLINTNEXTLINE(bugprone-*)
auto result = unsafe_func();

// 抑制區塊
// NOLINTBEGIN(readability-function-size)
void huge() { /* ... */ }
// NOLINTEND(readability-function-size)
```

[^clang-tidy-nolint]

## 7. 建議的最佳實踐

| 面向 | 建議 |
|------|------|
| 起手式 | 從已知風格（Google/LLVM）產生 `.clang-format`，再微調 |
| clang-tidy 檢查 | 初始先啟用 `bugprone-*`、`performance-*`、`modernize-*`，逐步擴充 |
| 編譯資料庫 | 永遠設定 `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` |
| CI 嚴格度 | CI 中設定 `WarningsAsErrors: '*'`，開發者可較寬鬆 |
| 團隊一致性 | 將 `.clang-format` 與 `.clang-tidy` 提交至版本管理 |
| 排除檔案 | 第三方或產生的程式碼使用 `.clang-format-ignore` 或 `SKIP_LINTING` 排除 |
| 版本鎖定 | 在 CI 與 pre-commit 中鎖定 Clang 工具版本，避免版本差異造成的結果不一致 |

## 總結

推薦的完整設定組合：

1. 在專案根目錄放置 **`.clang-format`** 與 **`.clang-tidy`** 設定檔
2. 在 **CMakeLists.txt** 中啟用 `CMAKE_CXX_CLANG_TIDY` 與 `CMAKE_EXPORT_COMPILE_COMMANDS`
3. 建立 **`format`** 與 **`format-check`** 自訂 CMake Target
4. 使用 **`cpp-linter/cpp-linter-hooks`** 作為 Pre-commit Hook
5. 在 GitHub Actions 中使用 **`cpp-linter-action`** 進行 CI 檢查

[^clang-format-official]: LLVM Project. (n.d.). ClangFormat. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormat.html
[^clang-tidy-official]: LLVM Project. (n.d.). Clang-Tidy. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/
[^clang-format-style]: LLVM Project. (n.d.). ClangFormat Style Options. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormatStyleOptions.html
[^clang-format-ignore]: LLVM Project. (n.d.). ClangFormat — .clang-format-ignore. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormat.html#clang-format-ignore
[^clang-tidy-config]: LLVM Project. (n.d.). Clang-Tidy — Configuration Files. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/#configuration-files
[^cmake-clang-tidy]: Sieger, D. (2021). Using clang-tidy with CMake. Retrieved 2026-09-25, from https://danielsieger.com/blog/2021/12/21/clang-tidy-cmake.html
[^cmake-lang-clang-tidy]: CMake. (n.d.). CMAKE_<LANG>_CLANG_TIDY. Retrieved 2026-09-25, from https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html
[^ttroy50-cmake-examples]: ttroy50. (n.d.). CMake Examples — clang-format. Retrieved 2026-09-25, from https://github.com/ttroy50/cmake-examples/tree/2b27fc75/04-static-analysis/clang-format
[^cpp-linter-hooks]: cpp-linter. (n.d.). cpp-linter-hooks. Retrieved 2026-09-25, from https://github.com/cpp-linter/cpp-linter-hooks
[^pre-commit-clang-format]: pre-commit. (n.d.). mirrors-clang-format. Retrieved 2026-09-25, from https://github.com/pre-commit/mirrors-clang-format
[^cpp-linter-action]: cpp-linter. (n.d.). cpp-linter-action. Retrieved 2026-09-25, from https://cpp-linter.github.io/cpp-linter-action/
[^clang-format-action]: jidicula. (n.d.). clang-format-action. Retrieved 2026-09-25, from https://github.com/marketplace/actions/clang-format-check
[^clang-format-commandline]: LLVM Project. (n.d.). ClangFormat — Standalone Tool. Retrieved 2026-09-25, from https://clang.llvm.org/docs/ClangFormat.html#standalone-tool
[^clang-tidy-commandline]: LLVM Project. (n.d.). Clang-Tidy — Command Line Options. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/
[^clang-tidy-nolint]: LLVM Project. (n.d.). Clang-Tidy — Suppressing Undesired Diagnostics. Retrieved 2026-09-25, from https://clang.llvm.org/extra/clang-tidy/#suppressing-undesired-diagnostics