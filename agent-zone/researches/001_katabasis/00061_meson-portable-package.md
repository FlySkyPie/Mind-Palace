# 使用 Meson 建置系統建立可攜式軟體套件

Meson 是一套現代化、高效能的建置系統，廣泛用於 GNOME、systemd、FFmpeg 等大型開源專案。本報告探討如何利用 Meson 的內建機制與生態工具，建立可移植（portable/relocatable）的軟體套件，適用於 AppImage、Flatpak、靜態連結二進位檔等場景。

---

## 1. Meson 內建的可攜式套件功能

### 1.1 可重定位的 pkg-config 檔案

Meson 的 **pkgconfig 模組** 從 0.63.0 版開始支援產生可重定位（relocatable）的 `.pc` 檔案，透過內建選項 `pkgconfig.relocatable` 啟用[^pkgc-reloc]：

```bash
meson setup builddir -Dpkgconfig.relocatable=true
```

啟用後，產生的 pkg-config 檔案中的 `prefix` 會改為**相對於 `install_dir`** 的路徑，而不是絕對路徑。這表示整個已安裝的套件可以任意移動到檔案系統的任何位置，只要相對路徑結構不變，依賴此套件的其他專案仍能正確找到它。若 `install_dir` 指向 prefix 外部，Meson 會回報錯誤[^pkgc-doc]。

### 1.2 Subproject 與 Wrap 依賴系統

Meson 的 Wrap 依賴系統專門為**自包含（self-contained）應用程式**而設計[^wrap]：

> 「在此類平台上，你必須產出自包含的應用程式。……傳統的做法是將依賴項捆綁到自己的專案中。……Meson 的 Wrap 依賴系統旨在提供自動化的方式來達成此目標。」

核心機制包含：

- **Wrap 檔案**（`.wrap`）：自動下載、驗證驗證碼並建置依賴項
- **WrapDB**：提供可直接使用的依賴配方（如 zlib、libpng、json-c 等）
- **Subproject**：巢狀的 Meson 專案，通常以靜態函式庫方式建置
- **修補覆蓋（Patch Overlays）**：可為沒有 Meson 建置定義的專案加入 Meson 支援

### 1.3 安裝標籤（install_tag）

從 0.60.0 版開始，Meson 支援 `install_tag`，允許選擇性地安裝檔案子集，對需要拆分套件的打包者非常實用[^install-tag]：

```meson
executable('prog', 'prog.c', install: true, install_tag: 'runtime')
install_headers('header.h', install_tag: 'devel')
```

安裝時可指定只安裝特定標籤：

```bash
meson install --tags runtime
```

預定義的標籤包含：`runtime`、`devel`、`doc`、`man`、`python-runtime`、`i18n`、`typelib`、`bin`、`bin-devel`、`tests`、`systemtap`。

---

## 2. `install()` 系列函式與套件安裝

### 2.1 基本安裝函式

Meson 提供多種安裝函式，覆蓋常見需求[^installing]：

```meson
# 執行檔
executable('prog', 'prog.c', install: true)

# 指定安裝目錄
executable('prog', 'prog.c', install: true, install_dir: 'my/special/dir')

# 標頭檔 → include/projname/header.h
install_headers('header.h', subdir: 'projname')

# 手冊頁 → share/man/man1/foo.1
install_man('foo.1')

# 資料檔 → share/progname/datafile.dat
install_data('datafile.dat', install_dir: get_option('datadir') / 'progname')

# 安裝時重新命名（0.46.0 起）
install_data('file.txt', rename: 'new-name.txt')

# 安裝整個子目錄樹
install_subdir('mydir', install_dir: 'include')

# 安裝到 prefix 外的絕對路徑（如 /etc）
install_data(sources: 'foo.dat', install_dir: '/etc')
```

### 2.2 自訂安裝腳本

從 0.38.0 版開始，可透過 `meson.add_install_script()` 加入自訂安裝腳本，處理複製檔案以外的邏輯[^ref-meson]：

```meson
meson.add_install_script('myscript.sh')
```

腳本可取的環境變數包含：

- `MESON_INSTALL_PREFIX` — 設定的安裝 prefix
- `MESON_INSTALL_DESTDIR_PREFIX` — DESTDIR 與 prefix 的串接結果
- `MESON_SOURCE_ROOT` — 原始碼目錄
- `MESON_BUILD_ROOT` — 建置目錄
- `MESON_INSTALL_QUIET` — 使用 `--quiet` 時設定
- `MESON_INSTALL_DRY_RUN` — 使用 `--dry-run` 時設定

### 2.3 授權清單安裝

```meson
meson.install_dependency_manifest('dependency-manifest.json')
```

此函式會安裝一份 JSON 清單，列出所有 subproject 的名稱、版本與授權資訊，對於散佈套件時的授權合規檢查非常有用[^ref-meson]。

---

## 3. 可重定位安裝（Prefix 相對路徑、RPATH 等）

### 3.1 Prefix 搭配 DESTDIR

Meson 的安裝路徑邏輯為：**`{DESTDIR}{prefix}{install_dir}`**。

若要建立可重定位的套件（如 AppDir），常見的做法是使用 `--prefix /` 搭配 `DESTDIR`[^prefix-issue]：

```bash
meson setup builddir --prefix /
DESTDIR=/path/to/AppDir meson install -C builddir
```

這樣會將檔案直接安裝到 `/path/to/AppDir/bin/`、`/path/to/AppDir/lib/` 等位置，避免產生巢狀 prefix 目錄。

### 3.2 RPATH 處理

`build_target()` 函式提供兩個 RPATH 相關關鍵字參數[^build-target]：

| 關鍵字 | 說明 |
|---|---|
| `build_rpath` | 在**建置目錄**中加到目標的 RPATH，安裝時會移除 |
| `install_rpath` | 安裝後設定為目標的 RPATH（Windows 無效） |

可重定位套件的關鍵設定：

```meson
executable('myapp', 'main.c',
    install: true,
    install_rpath: '$ORIGIN/../lib')
```

`$ORIGIN` 是 ELF 格式的特殊標記，代表執行檔自身所在目錄。設定 `$ORIGIN/../lib` 表示執行檔會在「自己所在目錄的上層的 lib 目錄」中尋找共享函式庫——無論套件被解壓縮到哪個路徑，這個相對關係都保持不變。

### 3.3 Meson 的自動 RPATH 行為

Meson 會自動偵測是否需要在建置階段加入 RPATH，以便開發者在建置目錄中直接執行測試。安裝時，這些**建置階段**的 RPATH 會被移除，僅套用 `install_rpath` 的設定值。

---

## 4. `meson install` 指令與打包工作流程

### 4.1 `meson install` 指令

從 0.47.0 版起，建議使用 `meson install` 取代舊式的 `ninja install`[^installing]：

```bash
meson install -C builddir
```

常用旗標：

| 旗標 | 說明 |
|---|---|
| `--destdir` | 同 DESTDIR 環境變數（0.57.0 起支援）；0.60.0 起支援相對路徑 |
| `--no-rebuild` | 安裝前不重新建置 |
| `--only-changed` | 僅安裝有變更的檔案 |
| `--tags` | 依標籤只安裝特定子集（0.60.0 起） |
| `--dry-run` | 僅顯示將安裝的檔案，不實際執行（1.1.0 起） |
| `--strip` | 安裝時去除除錯符號 |

### 4.2 DESTDIR 打包工作流程

標準的發行版打包流程[^quick-guide]：

```bash
# 1. 設定
meson setup builddir --prefix /usr --buildtype=plain \
    -Dc_args='...' -Dcpp_args='...'

# 2. 建置
meson compile -C builddir

# 3. 測試
meson test -C builddir

# 4. 安裝至暫存目錄
DESTDIR=/path/to/staging/root meson install -C builddir
```

使用 `--buildtype=plain` 會讓 Meson **不加入自己的編譯器旗標**，將控制權完全交給打包者。

### 4.3 `meson devenv`：不安裝的開發環境

開發者可以使用 `meson devenv` 在不安裝的情況下測試應用程式：

```bash
meson devenv -C builddir -c "./myapp"
```

此指令會設定 PATH、LD_LIBRARY_PATH 等環境變數，讓應用程式能找到建置目錄中的產物。

---

## 5. 以 Meson 散佈可攜式套件的最佳實踐

### 5.1 AppImage 搭配 linuxdeploy

透過 Meson 的 DESTDIR 支援與 linuxdeploy 工具，可以輕鬆建立 AppImage[^appimage]：

```bash
# 安裝到 AppDir
meson setup builddir --prefix /usr
DESTDIR=AppDir meson install -C builddir

# 捆綁依賴並產出 AppImage
linuxdeploy --appdir AppDir --output appimage
```

linuxdeploy 會自動將執行檔所需的共享函式庫複製到 AppDir 中，建立符號連結，最後產生 `.AppImage` 檔案。對於複雜的框架（如 Qt），可使用 linuxdeploy 的外掛系統（如 `--plugin qt`）。

### 5.2 靜態連結：Subproject 與 WrapDB

若要產出真正獨立的二進位檔（不依賴外部共享函式庫），可使用 Meson 的 subproject 系統搭配靜態連結[^wrap]：

```bash
meson setup builddir \
    --default-library=static \
    --force-fallback-for=dep1,dep2
```

或在 `meson.build` 中設定：

```meson
project('myapp', 'c', default_options: 'default_library=static')
```

WrapDB 提供多種常見依賴項（zlib、libpng、libdrm、json-c 等）的 Meson 建置配方，可作為 subproject 靜態連結。

### 5.3 使用 Zig CC 交叉編譯靜態二進位檔

將 **`zig cc`** 作為編譯器，可以產出完全靜態、跨平台編譯的二進位檔，且不依賴系統 C 函式庫（使用 musl libc）[^zig-meson]：

```bash
# Meson cross file: zig-aarch64.conf
cat > zig-aarch64.conf << 'EOF'
[binaries]
c = ['zig-wrapper.sh', 'aarch64-linux-musl']
ar = 'llvm-ar'
strip = 'llvm-strip'

[target_machine]
system = 'linux'
cpu_family = 'aarch64'
cpu = 'cortex-a53'
endian = 'little'
EOF

# 靜態連結 + 交叉編譯
meson setup builddir \
    --cross-file=zig-aarch64.conf \
    --default-library=static

meson compile -C builddir
```

透過此方式產出的 ARM64 Linux 二進位檔約 200KB，不依賴任何系統共用函式庫，可在任何 Linux 環境中執行。

### 5.4 Flatpak

Flatpak 原生支援 Meson 作為建置系統。使用 flatpak-builder 時，可直接指定 Meson 作為建置系統類型，無需額外設定[^flatpak]。

---

## 打包者檢查清單

綜合 Meson 官方文件與 Arch Linux 套件指南，打包可攜式套件時應注意以下事項[^quick-guide][^arch]：

1. **務必設定 `--prefix`** — 預設值為 `/usr/local`，對發行版套件不適用
2. **使用 `--buildtype=plain`** — 取得對編譯旗標的完整控制權
3. **使用 `DESTDIR` 進行暫存安裝** — 不要變造 `--prefix`
4. **啟用 `--strip`** — 安裝時去除除錯符號以縮小體積
5. **使用安裝標籤** — 使用 `--tags runtime`、`--tags devel` 等拆分套件
6. **啟用 Unity 建置**（`-Dunity=on`）— 從原始碼完整建置時加快速度
7. **使用 `--force-fallback-for`** — 強制對特定依賴使用 subproject
8. **設定 `install_rpath`** — 為可重定位套件設為 `$ORIGIN/../lib`
9. **啟用 `pkgconfig.relocatable`** — 散佈可重定位的 pkg-config 檔案
10. **使用 `install_dependency_manifest()`** — 確保套件中的授權合規

---

## 參考資料

[^pkgc-reloc]: Meson Build. (n.d.). Built-in options. Retrieved 2026-09-18, from https://mesonbuild.com/Builtin-options.html
[^pkgc-doc]: Meson Build. (n.d.). Pkgconfig module. Retrieved 2026-09-18, from https://mesonbuild.com/Pkgconfig-module.html
[^wrap]: Meson Build. (n.d.). Wrap dependency system manual. Retrieved 2026-09-18, from https://mesonbuild.com/Wrap-dependency-system-manual.html
[^installing]: Meson Build. (n.d.). Installing. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html
[^install-tag]: Meson Build. (n.d.). Install tags. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html#install-tags
[^ref-meson]: Meson Build. (n.d.). Reference manual — meson object. Retrieved 2026-09-18, from https://mesonbuild.com/Reference-manual_builtin_meson.html
[^build-target]: Meson Build. (n.d.). Reference manual — build_target functions. Retrieved 2026-09-18, from https://mesonbuild.com/Reference-manual_functions_build_target.html
[^prefix-issue]: Meson Build. (n.d.). GitHub issue #12880 — `meson install --prefix /` with DESTDIR. Retrieved 2026-09-18, from https://github.com/mesonbuild/meson/issues/12880
[^quick-guide]: Meson Build. (n.d.). Quick guide for distro packagers. Retrieved 2026-09-18, from https://mesonbuild.com/Quick-guide.html
[^appimage]: AppImage Documentation. (n.d.). Packaging guide — linuxdeploy user guide. Retrieved 2026-09-18, from https://docs.appimage.org/packaging-guide/from-source/linuxdeploy-user-guide.html
[^zig-meson]: Perez de Castro, G. (2023). Standalone binaries with zig cc and Meson. Retrieved 2026-09-18, from https://perezdecastro.org/2023/standalone-binaries-zigcc-meson.html
[^flatpak]: Flatpak Documentation. (n.d.). Build system — Meson. Retrieved 2026-09-18, from https://docs.flatpak.org/en/latest/meson.html
[^arch]: Arch Linux Wiki. (n.d.). Meson package guidelines. Retrieved 2026-09-18, from https://wiki.archlinux.org/title/Meson_package_guidelines