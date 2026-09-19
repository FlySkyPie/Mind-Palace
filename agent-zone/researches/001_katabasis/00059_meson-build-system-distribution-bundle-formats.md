# Meson 建置系統支援的發行／捆包格式

Meson 是一個開源建置系統，內建 `meson dist` 命令負責產生原始碼發行封存檔，同時也提供模組與腳本機制以擴充發行流程。

## 內建發行格式（meson dist）

`meson dist` 命令支援四種封存格式[^dist-cmd]：

| 格式名稱 | 副檔名 | 說明 |
|---|---|---|
| `xztar` | `.tar.xz` | **預設格式**，使用 XZ 壓縮 |
| `bztar` | `.tar.bz2` | 使用 BZip2 壓縮 |
| `gztar` | `.tar.gz` | 使用 GZip 壓縮 |
| `zip` | `.zip` | ZIP 格式 |

使用方式：

```bash
meson dist -C builddir
meson dist -C builddir --formats xztar,gztar,zip  # 同時產生多種格式
```

底層依賴 Python 標準庫 `shutil.make_archive()` 實作。

## 額外發行相關功能

### 1. dist script 自訂流程

透過 `meson.add_dist_script()` 在建立封存檔前執行自訂腳本，例如生成檔案、設定版本號、或呼叫外部打包工具[^create-releases]。

### 2. 子專案獨立發行

自 Meson 0.57.0 起，可在子專案目錄中獨立執行 `meson dist`，僅打包該子專案的原始碼[^create-releases]。

### 3. 包含 wrap 子專案

`--include-subprojects` 選項可將所有使用的 wrap 子專案一併納入發行封存檔，適合離線建置場景[^create-releases]。

### 4. 外部打包支援

Meson 本身**不內建** Flatpak、AppImage、Snap、RPM、DEB 等二進位發行格式，但可透過以下途徑達成：

- **Pkgconfig 模組**：產生 `.pc` 檔案，供套件管理系統使用[^modules]
- **GNOME 模組**：整合 AppStream、GIR、Typelib 等 GNOME 生態系元件[^modules]
- **install 命令**：將建置結果安裝至指定前綴，後續可搭配 `dpkg-deb`、`rpmbuild`、`flatpak-builder` 等外部工具進行打包

## 總結

| 層級 | 支援格式 | 方式 |
|---|---|---|
| 內建（原始碼） | `tar.xz`、`tar.bz2`、`tar.gz`、`zip` | `meson dist` 命令 |
| 腳本擴充 | 任意（可呼叫外部打包工具） | `meson.add_dist_script()` |
| 生態系整合 | pkg-config、AppStream、GIR 等 | 模組系統（pkgconfig、gnome 等） |
| 二進位發行 | 無內建，需搭配外部工具 | 自訂腳本 + 外部打包器 |

[^dist-cmd]: Meson 團隊. (n.d.). Command-line commands — `meson dist`. Retrieved 2026-09-18, from https://mesonbuild.com/Commands.html
[^create-releases]: Meson 團隊. (n.d.). Creating releases. Retrieved 2026-09-18, from https://mesonbuild.com/Creating-releases.html
[^modules]: Meson 團隊. (n.d.). Modules. Retrieved 2026-09-18, from https://mesonbuild.com/Modules.html