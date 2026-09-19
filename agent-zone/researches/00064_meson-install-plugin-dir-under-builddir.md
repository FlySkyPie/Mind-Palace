# Meson：如何將 Plugin 產出至 `builddir/` 下的自訂目錄結構

## 問題

希望在 `meson setup` 產生的 `builddir/` 下，直接產生一個立即可交付（ready to deliver）的 Plugin 目錄結構：

```
builddir/
  └── MyNativePlugin/
        ├── main.lua
        └── native.so
```

是否能透過覆寫 `prefix` 變數來達成？

## 結論：不可以覆寫 prefix，但有標準替代方案

Meson **不允許在 `meson install` 時覆寫 `prefix`**。prefix 在 `meson setup` 階段就固定了，無法在安裝時臨時更改[^reddit]。官方推薦的做法是使用 **DESTDIR** 或其 CLI 等效選項 `--destdir`[^meson-installing]。

## 四種可行作法

### 作法 A：`--destdir` + `prefix=/`（推薦，最簡潔）

將 prefix 設為 `/`，然後用 `--destdir` 將安裝根目錄指向 `builddir` 自身：

```bash
meson setup builddir --prefix=/
meson compile -C builddir
meson install -C builddir --destdir . --no-rebuild
```

由於 `meson.build` 中將檔案安裝到 `install_dir: 'MyNativePlugin'`，實際落點為：

```
builddir/MyNativePlugin/main.lua
builddir/MyNativePlugin/native.so
```

- `--destdir .`：自 Meson 0.60.0 起支援相對路徑（相對於 builddir）[^meson-commands]。
- `--no-rebuild`：跳過重新編譯，節省時間[^meson-installing]。
- 若不想污染 builddir，也可指向外部目錄：`--destdir /path/to/delivery`。

### 作法 B：安裝腳本（install script）

適用於需要複雜邏輯（如改名、壓縮）的情境。在 `meson.build` 中註冊自訂安裝腳本：

```meson
shared_module('native', 'native.c',
    install: true,
    install_dir: false  # 關閉預設安裝機制
)

meson.add_install_script('install_plugin.sh')
```

腳本內容範例[^meson-installing]：

```bash
#!/bin/sh
mkdir -p "${DESTDIR}/${MESON_INSTALL_PREFIX}/MyNativePlugin"
cp "${MESON_BUILD_ROOT}/libnative.so" \
   "${DESTDIR}/${MESON_INSTALL_PREFIX}/MyNativePlugin/native.so"
cp "${MESON_SOURCE_ROOT}/main.lua" \
   "${DESTDIR}/${MESON_INSTALL_PREFIX}/MyNativePlugin/main.lua"
```

### 作法 C：絕對路徑 `install_dir`

直接將安裝目錄設為絕對路徑，指向 builddir：

```meson
plugin_dir = meson.current_build_dir() / 'MyNativePlugin'

shared_module('native', 'native.c',
    install: true,
    install_dir: plugin_dir
)

install_data('main.lua',
    install_dir: plugin_dir
)
```

然後執行 `meson install -C builddir` 即可。缺點是 hardcode 了 builddir 路徑，降低可攜性。

### 作法 D：標準 prefix + DESTDIR（適合 packaging）

這是最常見的 packaging 流程，prefix 設為正常值，用 DESTDIR 暫存到 staging area：

```bash
meson setup builddir --prefix=/usr/local
meson compile -C builddir
DESTDIR=/tmp/staging meson install -C builddir
```

結果：

```
/tmp/staging/usr/local/MyNativePlugin/main.lua
/tmp/staging/usr/local/MyNativePlugin/native.so
```

可再打包成 tar.gz / deb / rpm。

## Meson Plugin 相關要點

### `shared_module()` vs `shared_library()`

- 若 `native.so` 是給其他程式用 `dlopen()` 動態載入的 Plugin，應使用 **`shared_module()`**（自 0.37.0 起）[^meson-shared-module]。
- `shared_module()` 預設不會加上 `lib` 前綴，產出直接是 `native.so`；而 `shared_library()` 會產出 `libnative.so`（除非設 `name_prefix: ''`）。

### `install_data()` 的目錄控制

- `install_dir` 支援相對路徑（相對於 prefix）與絕對路徑[^meson-install-data]。
- `preserve_path`（自 0.64.0 起）可保留來源目錄結構，相當於 GNU Automake 的 `nobase` 選項[^meson-install-data]。

## 總結對照表

| 作法 | 指令 | 產出路徑 | 適合場景 |
|------|------|----------|----------|
| A：`--destdir` + `prefix=/` | `meson install -C builddir --destdir .` | `builddir/MyNativePlugin/...` | 開發階段快速交付 |
| B：安裝腳本 | `meson.add_install_script()` | 自訂 | 複雜邏輯、壓縮、改名 |
| C：絕對路徑 install_dir | `install_dir: meson.current_build_dir() / 'X'` | `builddir/MyNativePlugin/...` | 簡單但可攜性差 |
| D：標準 DESTDIR staging | `DESTDIR=/tmp/staging meson install` | `/tmp/staging/...` | 正式 packaging |

## 參考文獻

[^reddit]: Reddit r/meson. (2019). *How to override prefix during installation?* Retrieved 2026-09-19, from https://www.reddit.com/r/meson/comments/a9lcwx/how_to_override_prefix_during_installation/

[^meson-installing]: The Meson Build System. (n.d.). *Installing*. Retrieved 2026-09-19, from https://mesonbuild.com/Installing.html

[^meson-commands]: The Meson Build System. (n.d.). *Command-line commands*. Retrieved 2026-09-19, from https://mesonbuild.com/Commands.html

[^meson-shared-module]: The Meson Build System. (n.d.). *shared_module()*. Retrieved 2026-09-19, from https://mesonbuild.com/Reference-manual_functions_shared_module.html

[^meson-install-data]: The Meson Build System. (n.d.). *install_data()*. Retrieved 2026-09-19, from https://mesonbuild.com/Reference-manual_functions_install_data.html