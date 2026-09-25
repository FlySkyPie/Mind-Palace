# Meson Build System：開發者工具與公用設施的建置支援

## 摘要

本文探討 Meson 建置系統是否能夠處理「開發者工具（developer tools）」——即那些不直接被原始碼使用、但對開發者有幫助的工具（如 linter、formatter、程式碼產生器、文件產生器等）。研究結果顯示 Meson 提供了完整且優雅的內建機制來支援此類需求，尤其在跨編譯情境下處理主機（host machine）工具的能力是其一大亮點。

---

## 1. 問題定義

在 C/C++ 專案中，開發者常會使用以下工具來提升生產力與程式碼品質：

- **程式碼風格檢查與格式化工具**：clang-format、clang-tidy、cppcheck
- **程式碼產生器**：protobuf compiler、Qt MOC/uic/rcc、IDL compiler
- **文件產生器**：Doxygen、Sphinx
- **基準測試工具**：benchmark runner
- **靜態分析工具**：Coverity、SonarQube

這些工具並非最終產品的直接依賴，但對開發過程至關重要。問題在於：Meson 能否以優雅、慣例化的方式將這些工具整合進建置流程？

---

## 2. Meson 的內建機制

Meson 提供了多層次的基元（primitives）來滿足不同類型的開發者工具需求：[^meson-refman]

### 2.1 `run_target()` —— 無輸出檔案的副作用開發任務

`run_target()` 建立一個頂層目標，執行指定的命令，但「就 Meson 的觀點而言不產生任何輸出」[^run-target]：

```meson
run_target('format',
  command : ['clang-format', '-i', '-style=file', sources])

run_target('tidy',
  command : ['run-clang-tidy.py', '-fix', '-j', '8', sources])
```

- 透過 `meson compile format` 或 `ninja format` 呼叫
- 支援 `depends`（先建置其他目標）、`env`（環境變數）以及模板字串替換（`@SOURCE_ROOT@`、`@BUILD_ROOT@`、`@CURRENT_SOURCE_DIR@`）
- **適用場景**：linter、formatter、韌體燒錄、benchmark runner、文件產生

### 2.2 `custom_target()` —— 會產生輸出檔案的程式碼產生器

`custom_target()` 建立一個自訂建置目標，具有明確的 `input`、`output` 和 `command` 參數：[^custom-target]

```meson
mygen = executable('mygen', 'tools/mygen.c', native : true)

generated = custom_target('gen-src',
  input : ['input.idl'],
  output : ['generated.c', 'generated.h'],
  command : [mygen, '@INPUT@', '@OUTPUT0@', '@OUTPUT1@'])
```

- 支援豐富的模板替換：`@INPUT@`、`@OUTPUT0@`、`@PLAINNAME@`、`@BASENAME@`、`@OUTDIR@`、`@DEPFILE@` 等
- 可以索引化取用個別輸出檔案（如 `foo_ch[1]` 代表第一個產生的 header）
- **關鍵旗標**：
  - `build_always_stale : true` —— 永遠視為過期，即使輸出檔案已存在（適合建置時間戳記、版本號產生）
  - `build_by_default : true` —— 加入預設 `meson compile` 呼叫
  - `console : true` —— 用於長時間、資源密集的命令（使用 Ninja 的 `console` pool）
- **適用場景**：IDL 編譯器、protobuf 程式碼產生器、Qt 的 MOC/uic/rcc

### 2.3 `alias_target()` —— 目標分組（自 0.52.0 版起）

一個不做任何事但確保所有依賴目標都被建置的虛擬目標：[^alias-target]

```meson
alias_target('check-all', 'test', 'tidy', 'format')
```

- 自 0.60.0 版起支援包含 `run_target` 的依賴
- **適用場景**：建立便利目標如 `meson compile check-all` 來同時執行測試、linter 和程式碼格式化

### 2.4 `generator()` —— 批量檔案轉換

`generator()` 定義如何將輸入檔案轉換為一個或多個輸出檔案，然後透過 `gen.process()` 應用至多個檔案：[^generator]

```meson
idl_gen = generator(idl_compiler,
  output : '@BASENAME@.h',
  arguments : ['@INPUT@', '@OUTPUT@'])

generated_headers = idl_gen.process(['input1.idl', 'input2.idl'])
```

- **限制**：僅能應用於單一 build target。若多個目標需要輸出，應使用 `custom_target()`

### 2.5 `run_command()` —— 配置階段的檢查（非建置階段）

`run_command()` 在 `meson setup`（配置階段）執行命令，而非建置階段。返回 `runresult` 物件：[^run-command]

```meson
git_hash = run_command('git', 'rev-parse', 'HEAD').stdout().strip()
```

- **適用場景**：偵測系統能力、產生配置標頭、在建置前檢查工具版本

### 2.6 功能比較摘要

| 功能 | 執行時機 | 產生輸出？ | 典型用途 |
|---|---|---|---|
| `run_target()` | 建置時（`meson compile`） | 否（僅副作用） | linter、formatter、燒錄 |
| `custom_target()` | 建置時 | 是（明確輸出） | 程式碼產生、文件產生 |
| `generator()` | 建置時 | 是（每個輸入產生輸出） | 批量轉換（IDL → 原始碼） |
| `alias_target()` | 建置時（作為依賴） | 否 | 目標分組 |
| `run_command()` | 配置時（`meson setup`） | 捕獲 stdout | 系統偵測、配置檢查 |

---

## 3. 跨編譯情境：同時為建置機器與目標機器建置工具

這是 Meson 最強大的功能之一。跨編譯時，開發者工具需要在**建置機器**（build machine，正在編譯的那台電腦）上執行，而非**目標機器**（host machine，程式最終運行的環境）。Meson 透過 `native : true` 關鍵字參數優雅地解決這個問題：[^cross-compile]

```meson
native_exe = executable('mygen', 'mygen.c', native : true)
```

指定 `native : true` 後，Meson 使用建置機器的原生編譯器（而非交叉編譯器）來編譯 `mygen`。接著可以將 `native_exe` 直接用於 `custom_target()` 或 `generator()` 的命令中，產生的程式碼再編譯給目標機器使用。

### 3.1 輔助基礎設施

- **`meson.is_cross_build()`** —— 返回 `true` 代表正在進行交叉編譯，可用於條件式設定
- **`meson.can_run_host_binaries()`** —— 若已配置 `exe_wrapper`（如 Wine、QEMU），則返回 `true`
- **`find_program(..., native : true)`** —— 在建置機器上搜尋程式，而非目標機器
- **`get_compiler('c', native : true)`** —— 取得建置機器的編譯器
- **Native files**（`--native-file` 傳入的 `.ini` 檔案）—— 描述建置機器環境，類似於 cross files 之於目標機器

### 3.2 完整範例：跨編譯安全的程式碼產生

```meson
project('my-project', 'c')

# 為建置機器編譯程式碼產生器
mygen = executable('mygen', 'tools/mygen.c', native : true)

# 使用該產生器為目標機器產生原始碼
generated_sources = custom_target('protobuf-gen',
  input : ['schema.proto'],
  output : ['schema.pb.c', 'schema.pb.h'],
  command : [mygen, '--proto', '@INPUT@',
             '--output-c', '@OUTPUT0@',
             '--output-h', '@OUTPUT1@'])

# 編譯產生的原始碼給目標機器
main_lib = library('mylib',
  sources : ['src/core.c', generated_sources])
```

這是 Meson 慣例化的跨編譯開發工具處理方式，僅需一個 `native : true` 關鍵字，無需額外的建置系統呼叫或複雜的 `ExternalProject` 設定。

---

## 4. 慣例化的專案結構

參考 [meson-sample-project](https://github.com/tiernemi/meson-sample-project) 及 GNOME、systemd、QEMU 等真實專案的做法，建議結構如下：[^sample-project]

```
my-project/
├── meson.build              # 頂層：project(), subdir(), 頂層目標
├── meson_options.txt        # 選項如 'enable-tests', 'enable-docs'
├── src/                     # 主要原始碼
│   └── meson.build
├── include/                 # 公開標頭檔
│   └── meson.build
├── tools/                   # 開發者工具原始碼（可選）
│   └── meson.build
├── tests/                   # 測試（依選項條件編譯）
│   └── meson.build
├── benchmarks/              # 基準測試（依選項條件編譯）
│   └── meson.build
├── docs/                    # 文件（如 Doxygen 配置）
│   └── meson.build
├── subprojects/             # Wrap 依賴
│   ├── fmt.wrap
│   └── gtest.wrap
└── data/                    # 要安裝的資料/設定檔
```

### 4.1 透過選項條件式啟用開發工具

```meson
# meson_options.txt
option('enable-tests', type : 'boolean', value : true)
option('enable-docs', type : 'boolean', value : true)

# meson.build
if get_option('enable-tests')
  subdir('tests')
endif
if get_option('enable-docs')
  subdir('docs')
endif
```

### 4.2 頂層便利目標

```meson
# 在 root meson.build 中
clang_format = find_program('clang-format', required : false)
if clang_format.found()
  run_target('format',
    command : [clang_format, '-i', '-style=file', sources])
endif

run_target('tidy',
  command : [find_program('run-clang-tidy.py'), '-fix', sources])

alias_target('check-all', 'test', 'tidy', 'format')
```

### 4.3 配置時摘要

```meson
summary({'Tests' : get_option('enable-tests'),
         'Benchmarks' : get_option('enable-benchmarks'),
         'Documentation' : get_option('enable-docs')},
        section : 'Developer Tooling')
```

---

## 5. 與其他建置系統的比較

| 面向 | Meson | CMake | Bazel |
|---|---|---|---|
| **Linter/Formatter 目標** | `run_target('tidy', command : [...])` — 簡單、一等公民、無需輸出檔案 | `add_custom_target(tidy COMMAND ...)` — 類似但語法較冗長；預設永遠被視為過期 | `sh_binary` 或自訂 Starlark rule；更具封閉性但語法更繁瑣 |
| **程式碼產生（含輸出）** | `custom_target()` 搭配豐富模板語法（`@INPUT@`、`@OUTPUT0@`、`@PLAINNAME@`）。`generator()` 用於批量處理 | `add_custom_command(OUTPUT ... COMMAND ...)` — 需要額外 target 驅動；樣板程式碼較多 | `genrule()` 或自訂 `rule()` — 功能強大但學習曲線陡峭 |
| **跨編譯的主機工具** | `executable('tool', 'tool.c', native : true)` — 一個關鍵字。第一等支援，優雅 | `ExternalProject_Add` 或獨立 `CMake` 呼叫；無等效 `native` 關鍵字 | `cfg = "host"` / `cfg = "exec"` — 明確但需理解 Bazel 的配置系統 |
| **群組目標** | `alias_target('all-checks', 'test', 'tidy')` — 自 0.52.0 起 | 自訂目標搭配 `ALL` 選項；無專用 alias 概念 | `alias()` — 類似概念 |
| **配置時腳本** | `run_command()` — 在 `meson setup` 時執行，返回 `runresult` 物件 | `execute_process()` — 類似，在 configure 時執行 | 無法輕鬆在 configure 時執行任意命令；依賴 repository rules |
| **使用難易度** | 學習曲線低；Python 風格 DSL；非圖靈完備（易於理解與維護） | 中高級；語法複雜且有歷史包袱；圖靈完備（功能強大但痛苦） | 高；需學習 Starlark、toolchains 及 workspace 概念；前期投資重大 |
| **生態系統** | 較小但持續成長；wrap file 系統管理依賴；在 GNOME/GStreamer/Freedesktop 專案中表現優異 | 最大生態系統；幾乎所有 C++ 函式庫都有 CMake 支援；大量 CI 範例 | 以 Google 為中心；適合 monorepo；開源支援逐漸成長 |

### 5.1 哲學差異

- **Meson**：將開發工具視為**第一等目標**（`run_target`、`custom_target`），使用簡潔可讀的 DSL。跨編譯主機工具的 `native: true` 只需一個關鍵字，無需多餘儀式。非圖靈完備的 DSL 讓建置檔案易於維護。

- **CMake**：具備等效能力（`add_custom_target`、`add_custom_command`、`execute_process`），但語法更冗長且容易出錯。跨編譯主機工具**沒有內建的 `native` 等效機制**——通常需要 `ExternalProject` 或第二次 CMake 呼叫，顯著增加複雜度。

- **Bazel**：將**封閉性與再現性**置於最高優先級。開發工具通常以 `genrule()` 或 `sh_binary` 目標表達，並受益於 Bazel 的遠端快取與執行。然而學習曲線陡峭，對中小型專案而言，Bazel 的 workspace 設定負擔往往超過其帶來的效益。

---

## 6. 結論

**是的，Meson 完全可以處理開發者工具的建置需求。**

即使工具不直接被原始碼使用——如 linter、formatter、程式碼產生器、文件產生器等——Meson 也提供了完整且優雅的內建支援：

1. **`run_target()`**：專門用於無輸出檔案的開發任務（linter、formatter），語法簡潔
2. **`custom_target()`**：用於有明確輸出的工具（程式碼產生器、文件產生器），支援豐富的模板替換
3. **`alias_target()`**：可將多個開發任務群組成單一便利目標
4. **`native : true`**：跨編譯時只需一個關鍵字即可在建置機器上編譯工具，再用該工具為目標機器產生程式碼

在跨編譯支援方面，Meson 明顯優於 CMake（後者缺乏等效的簡單機制），而在易用性方面又勝過 Bazel。對於重視**建置腳本可讀性、快速配置時間、以及一流的開發工具與跨編譯支援**的專案，Meson 是一個絕佳的選擇。

---

[^meson-refman]: Meson build system. (n.d.). *Reference manual*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual.html
[^run-target]: Meson build system. (n.d.). *run_target()*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_run_target.html
[^custom-target]: Meson build system. (n.d.). *custom_target()*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_custom_target.html
[^alias-target]: Meson build system. (n.d.). *alias_target()*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_alias_target.html
[^generator]: Meson build system. (n.d.). *generator()*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_generator.html
[^run-command]: Meson build system. (n.d.). *run_command()*. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_run_command.html
[^cross-compile]: Meson build system. (n.d.). *Cross compilation*. Retrieved 2026-09-25, from https://mesonbuild.com/Cross-compilation.html
[^sample-project]: tiernemi. (n.d.). *meson-sample-project*. GitHub. Retrieved 2026-09-25, from https://github.com/tiernemi/meson-sample-project