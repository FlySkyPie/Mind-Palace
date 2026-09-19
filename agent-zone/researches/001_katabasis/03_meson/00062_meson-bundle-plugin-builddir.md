# 使用 Meson 在建置目錄（builddir）下組合可交付 Plugin 套件

目標：在建置目錄 `builddir/` 下產生可直接交付的 Plugin 資料夾結構：

```
builddir/
  MyNativePlugin/
    ├── main.lua
    └── native.so
```

## 背景

Cuberite Plugin 的標準結構要求 Plugin 資料夾內包含 `main.lua` 與動態函式庫（`.so`）[^cuberite-plugin-struct]：

```
Server/Plugins/MyNewPlugin/
  ├── main.lua
  ├── Info.lua
  └── ...（其他 .lua、.so）
```

Cuberite 在 Plugin 載入時會將 Plugin 資料夾加入 `package.cpath`，搜尋模式為 `?.so`，因此放在 Plugin 資料夾根目錄的 `.so` 可直接被 `require()` 載入[^cuberite-so-loading]。

## Meson 方案

`custom_target()` 的 `build_subdir` 關鍵字參數可直接將輸出放入指定子目錄，但此功能**自 Meson 1.10.0 才支援**[^build-subdir]。若使用 1.3.2 等較舊版本，需透過 `sh -c` 或腳本手動組裝目錄。

### 方案一：內嵌 `sh -c` 指令（無需額外腳本）

適用於只需要組合少量檔案的簡單情況。

```meson
# meson.build (at project root)
project('MyNativePlugin', 'c', version: '1.0.0')

# 建置 native.so（不安裝到系統 prefix）
native_so = shared_library('native', 'native.c', install: false)

# 將 native.so 與 main.lua 組合成可交付的 Plugin 目錄
custom_target('bundle_MyNativePlugin',
  input: [native_so, files('main.lua')],
  output: '.bundle_stamp',
  command: [
    'sh', '-c', '''
      mkdir -p MyNativePlugin &&
      cp "@INPUT0@" MyNativePlugin/native.so &&
      cp "@INPUT1@" MyNativePlugin/main.lua &&
      touch "@OUTPUT@"
    ''',
    '_',                     # $0 (placeholder for argv[0])
    '@INPUT0@',              # $1 = native.so 完整路徑
    '@INPUT1@',              # $2 = main.lua 完整路徑
    '@OUTPUT@'               # $3 = .bundle_stamp 完整路徑
  ],
  build_by_default: true)    # meson compile 時自動執行
```

結果：執行 `meson compile -C builddir` 後，`builddir/MyNativePlugin/` 會包含所需的兩個檔案。

### 方案二：獨立腳本（適合複雜組合邏輯）

建立 `tools/bundle-plugin.sh`：

```bash
#!/bin/sh
set -e
output_dir="$1"
native_so_src="$2"
lua_src="$3"

mkdir -p "${output_dir}/MyNativePlugin"
cp "${native_so_src}" "${output_dir}/MyNativePlugin/native.so"
cp "${lua_src}"       "${output_dir}/MyNativePlugin/main.lua"
```

meson.build 中引用腳本：

```meson
bundle_script = find_program('tools/bundle-plugin.sh')

custom_target('bundle_MyNativePlugin',
  input: [native_so, files('main.lua')],
  output: '.bundle_stamp',
  command: [
    bundle_script,
    '@OUTDIR@',     # 建置目錄路徑（builddir/）
    '@INPUT0@',     # native.so 完整路徑
    '@INPUT1@'      # main.lua 完整路徑
  ],
  build_by_default: true)
```

### 方案三：安裝至 DESTDIR（適合正式發布）

若 Plugin 安裝目標是系統 prefix（如 `/usr/local`），可使用標準安裝機制：

```meson
shared_library('native', 'native.c', install: true, install_dir: 'MyNativePlugin')
install_data('main.lua', install_dir: 'MyNativePlugin')
```

安裝指令：

```bash
DESTDIR=/path/to/packaging/root meson install -C builddir
```

結果：檔案被安裝到 `/path/to/packaging/root/prefix/MyNativePlugin/`[^meson-install-destdir]。

## 關鍵參數說明

| 函式 | 關鍵字 | 用途 |
|------|--------|------|
| `custom_target()` | `output: '.bundle_stamp'` | 佔位輸出檔，代表組合已完成 |
| `custom_target()` | `build_by_default: true` | 讓 `meson compile`（不帶引數）時自動執行 |
| `shared_library()` | `install: false` | 不安裝到系統 prefix（僅存在於建置目錄） |
| `find_program()` | — | 尋找專案內的自訂腳本 |
| `@OUTDIR@` | — | `custom_target` 內建變數，指 output 所在目錄（即 `builddir/`） |
| `@INPUT0@`, `@INPUT1@` | — | `custom_target` 內建變數，依序對應 input 陣列元素 |

## 注意事項

- `custom_target()` 的 `output` 檔案名稱**不可包含路徑分隔符**（`/` 或 `\`）。因此使用佔位檔 `.bundle_stamp` 並在指令中手動建立目標目錄[^custom-target]。
- 方案一中的 `'_'` 是 `sh -c` 語法要求的參數佔位，對應 `$0`，不影響實際功能。
- 若專案中有多個 Plugin，建議用 `subdir()` 或不同 `build_subdir` 來隔離目錄。

## 參考資料

[^cuberite-plugin-struct]: Cuberite Contributor. (n.d.). *Cuberite Plugin 資料夾結構與安裝方式*. Retrieved 2026-09-18, from `agent-zone/researches/001_katabasis/00054_cuberite-plugin-folder-structure-and-installation.md`

[^cuberite-so-loading]: Cuberite Contributor. (n.d.). *Cuberite Lua Plugin 能否載入動態函式庫（.so）*. Retrieved 2026-09-18, from `agent-zone/researches/001_katabasis/00039_cuberite-lua-plugin-dot-so-loading-zh.md`

[^build-subdir]: Meson Team. (n.d.). *Reference Manual — custom_target()*. Retrieved 2026-09-18, from https://mesonbuild.com/Reference-manual_functions.html#custom_target

[^custom-target]: Meson Team. (n.d.). *Custom Build Targets*. Retrieved 2026-09-18, from https://mesonbuild.com/Custom-build-targets.html#details-on-command-invocation

[^meson-install]: Meson Team. (n.d.). *Installing*. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html

[^meson-install-destdir]: Meson Team. (n.d.). *Installing — DESTDIR support*. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html#destdir-support