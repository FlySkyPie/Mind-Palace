# Meson Subproject 能否管理開發者工具？

## 摘要

Meson 建置系統雖然沒有像 npm 的 `devDependencies` 那樣專用的「僅開發用」依賴標記，但透過 `run_target()`、`find_program()` 與 `[provide] program_names` 等機制的組合，完全可以勝任管理開發者工具（如 linter、formatter、程式碼產生器）的需求。本報告詳細說明各種實作模式與最佳實踐。

## 問題陳述

在使用 Meson 的專案中，有些工具並非最終建置產出的一部分，也不會被原始碼直接連結或編譯，但對開發者而言非常有用——例如 clang-format、clang-tidy、客製化 linter、文件產生器、或程式碼產生器等。問題是：Meson 的 subproject 機制能否有效地管理這些工具？

## 核心機制：`run_target()`

`run_target()` 是 Meson 專門為此需求設計的機制。根據官方文件：

> Sometimes you need to have a target that just runs an external command. As an example you might have a build target that reformats your source code, runs `cppcheck` or something similar.[^run-targets]

`run_target()` 的關鍵特性：
- **預設不會在建置時執行**——只有在明確呼叫時才會執行（`meson compile <target_name>`）
- 對 Meson 而言**不產生任何輸出檔案**
- 與後端整合（例如 Ninja 使用者可以用 `ninja format` 觸發）

```meson
run_target('inspector',
  command : ['scripts/inspect.sh', '--exclude', 'tests'])
```

[^run-targets]: The Meson Build System. (n.d.). Run targets. Retrieved 2026-09-25, from https://mesonbuild.com/Run-targets.html

## Subproject 提供可執行檔：`[provide] program_names`

Meson 的 wrap 檔案可以在 `[provide]` 區段宣告「這個 subproject 會提供某個可執行檔」。搭配 `meson.override_find_program()`，主專案只要使用 `find_program()` 就會自動回退到 subproject 中建置的版本。[^wrap-provide]

```ini
# subprojects/my-tool.wrap
[wrap-git]
url = https://github.com/example/my-tool.git
revision = v1.0.0
depth = 1

[provide]
program_names = my-tool
```

```meson
# subprojects/my-tool/meson.build
project('my-tool', 'c')
my_tool_exe = executable('my-tool', 'main.c', install: true)
meson.override_find_program('my-tool', my_tool_exe)
```

```meson
# 主專案 meson.build
project('my-project', 'c')
my_tool = find_program('my-tool', required: get_option('my_tool'))
if my_tool.found()
  run_target('run-my-tool', command: [my_tool, '--do-something'])
endif
```

[^wrap-provide]: The Meson Build System. (n.d.). Wrap dependency system manual. Retrieved 2026-09-25, from https://mesonbuild.com/Wrap-dependency-system-manual.html

## 類比 npm `devDependencies` 的模式

Meson 沒有直接的 `devDependencies` 等效關鍵字，但可透過以下模式達到相同效果：

### 模式 A：Feature Options 條件啟用

使用 Meson 的 `feature` 選項類型（支援 `enabled`/`disabled`/`auto` 三態）：[^build-options]

```meson
# meson_options.txt
option('linter', type: 'feature', value: 'auto',
       description: 'Enable linter checks')
```

使用者可以控制：
```sh
meson setup builddir -Dlinter=enabled    # 強制啟用
meson setup builddir -Dlinter=disabled   # 強制關閉
meson setup builddir -Dlinter=auto       # 系統預設
```

### 模式 B：Optional `find_program` 優雅降級

這是最接近 npm `devDependencies` 的做法——工具如果能找到就用，找不到也不阻礙建置：[^find-program]

```meson
clang_format = find_program('clang-format', required: false)
if clang_format.found()
  run_target('format', command : [clang_format, '-i', sources])
endif
```

### 模式 C：總開關選項

```meson
option('dev_tools', type: 'boolean', value: true,
       description: 'Build and install developer tools')
```

```meson
if get_option('dev_tools')
  subproject('code-generator')
  subproject('doc-generator')
endif
```

[^build-options]: The Meson Build System. (n.d.). Build options. Retrieved 2026-09-25, from https://mesonbuild.com/Build-options.html
[^find-program]: The Meson Build System. (n.d.). find_program. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_find_program.html

## 各機制的用途對照

| 機制 | 何時執行 | 用途 | 連結到建置輸出？ |
|---|---|---|---|
| `run_target()` | 僅手動呼叫 | Linter、formatter、文件產生、程式碼分析 | 否 |
| `custom_target()` | 每次建置（如果輸入有變） | 產生需要編譯的原始碼 | 是 |
| `meson.add_dist_script()` | `meson dist` 期間 | 產生 changelog、版本檔或釋出用文件 | 否 |
| `meson.add_install_script()` | `meson install` 期間 | 執行安裝後步驟 | 否 |
| `meson.add_postconf_script()` | `meson setup` 完成後 | 產生設定檔、下載資源 | 否 |
| `meson.add_devenv()` | `meson devenv` 使用時 | 設定 PATH、函式庫路徑 | 否 |
| 內建 `clang-format` | 手動呼叫 | 格式化 C/C++ 程式碼（自動偵測） | 否 |

## 內建 clang-format 支援

自 Meson 0.50.0 起，系統如果安裝了 `clang-format` 且在專案根目錄找到 `.clang-format` 檔案，Meson 會自動加入 `clang-format` target：[^code-formatting]

```sh
ninja -C builddir clang-format
```

另有 `clang-format-check` target 可供 CI 使用。

[^code-formatting]: The Meson Build System. (n.d.). Code formatting. Retrieved 2026-09-25, from https://mesonbuild.com/Code-formatting.html

## 開發者環境：`meson devenv`

`meson.add_devenv()` 搭配 `meson devenv` 可以建構一個完整的開發者環境，讓 subproject 中建置的工具自動出現在 PATH 中：[^devenv]

```meson
devenv = environment()
devenv.set('PATH', meson.current_build_dir(), separator: ':')
devenv.prepend('LD_LIBRARY_PATH', meson.current_build_dir())
meson.add_devenv(devenv)
```

開發者只需：
```sh
meson devenv -C builddir
# 現在 subproject 建置的所有工具都在 PATH 中
```

[^devenv]: The Meson Build System. (n.d.). meson.add_devenv. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_builtin_meson.html#mesonadd_devenv

## Wrap 檔案機制

Meson 支援兩種 wrap 檔案：[^wrap-manual]

**`wrap-file`** — 下載原始碼壓縮檔：
```ini
[wrap-file]
directory = libfoobar-1.0
source_url = https://example.com/foobar-1.0.tar.gz
source_filename = foobar-1.0.tar.gz
source_hash = 5ebeea0dfb75d090ea0e7ff84799b2a7a1550db3fe61eb5f6f61c2e971e57663

[provide]
program_names = foobar-tool
```

**`wrap-git`** — 複製 Git 倉庫：
```ini
[wrap-git]
url = https://github.com/libfoobar/libfoobar.git
revision = HEAD
depth = 1

[provide]
program_names = foobar-tool
```

`meson subprojects` 相關指令：
```sh
meson subprojects download     # 擷取所有 subproject
meson subprojects update       # 更新至最新版本
meson subprojects checkout     # 跨 subproject 管理分支
meson subprojects foreach <cmd>  # 在每個 subproject 目錄執行指令
```

### `--wrap-mode` 選項

| 模式 | 行為 |
|---|---|
| `--wrap-mode=nodownload` | 禁止網路存取，僅使用已存在的原始碼 |
| `--wrap-mode=nofallback` | 不使用 subproject 回退，僅查詢系統 |
| `--wrap-mode=forcefallback` | 強制使用所有 subproject 回退，忽略系統 |
| `--force-fallback-for=list` | 僅對特定依賴/程式強制回退 |
| `--wrap-mode=nopromote` | 不自動複製 wrap 檔案（自 0.56.0） |

[^wrap-manual]: The Meson Build System. (n.d.). Wrap dependency system manual. Retrieved 2026-09-25, from https://mesonbuild.com/Wrap-dependency-system-manual.html

## 最佳實踐總結

1. **僅供開發者手動使用的工具**（formatter、linter）→ 使用 `run_target()` 搭配 `find_program(..., required: false)`
2. **需要從原始碼編譯的客製工具** → 使用 wrap 檔案的 `[provide] program_names` 搭配 `meson.override_find_program()`
3. **使用者可選擇是否啟用的工具** → 使用 `feature` 類型選項（`auto`/`enabled`/`disabled`）
4. **需要完整開發環境的工具** → 使用 `meson.add_devenv()` + `meson devenv`
5. **產生編譯用原始碼的工具**（code generator）→ 使用 `custom_target()`，這不是「純開發工具」

## 結論

**可以。** Meson 完全能夠勝任管理開發者工具的需求。雖然沒有專屬的 `devDependencies` 關鍵字，但 `run_target()`、`find_program()`、`[provide] program_names`、`meson.override_find_program()` 以及 feature option 的組合涵蓋了所有使用場景。推薦的做法是：為工具建立 wrap 檔並宣告 `[provide] program_names`，在 subproject 中呼叫 `meson.override_find_program()`，然後在主專案中用 `find_program()` 搭配 feature option 決定是否啟用，最後用 `run_target()` 讓開發者可手動呼叫。

## 參考來源

- The Meson Build System. (n.d.). Subprojects. Retrieved 2026-09-25, from https://mesonbuild.com/Subprojects.html
- The Meson Build System. (n.d.). Run targets. Retrieved 2026-09-25, from https://mesonbuild.com/Run-targets.html
- The Meson Build System. (n.d.). Wrap dependency system manual. Retrieved 2026-09-25, from https://mesonbuild.com/Wrap-dependency-system-manual.html
- The Meson Build System. (n.d.). find_program. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions_find_program.html
- The Meson Build System. (n.d.). Built-in meson object. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_builtin_meson.html
- The Meson Build System. (n.d.). Code formatting. Retrieved 2026-09-25, from https://mesonbuild.com/Code-formatting.html
- The Meson Build System. (n.d.). Creating releases. Retrieved 2026-09-25, from https://mesonbuild.com/Creating-releases.html
- The Meson Build System. (n.d.). Build options. Retrieved 2026-09-25, from https://mesonbuild.com/Build-options.html
- The Meson Build System. (n.d.). Custom build targets. Retrieved 2026-09-25, from https://mesonbuild.com/Custom-build-targets.html