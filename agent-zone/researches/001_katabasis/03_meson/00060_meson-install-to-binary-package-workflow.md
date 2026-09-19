# Meson 安裝命令與二進位打包流程

`meson install` 命令將建置結果安裝至指定前綴（prefix），結合 `DESTDIR` 機制可安裝至暫存區，再由外部打包工具（`dpkg-deb`、`rpmbuild`、`flatpak-builder`）製成二進位套件。本文說明各環節的具體作法。

## 共通基礎：meson install + DESTDIR

`meson install` 安裝至 `--prefix` 指定的目錄（預設 `/usr/local`）。若要將檔案「裝進暫存目錄」而非最終位置，使用 `DESTDIR`[^install]:

```bash
meson setup builddir --prefix /usr
meson compile -C builddir

# 安裝至暫存區
DESTDIR=/path/to/staging meson install -C builddir
# 或等同寫法（Meson ≥0.57.0）
meson install -C builddir --destdir /path/to/staging
```

暫存區的目錄結構會是 `/path/to/staging/usr/bin/...`、`/path/to/staging/usr/lib/...`，打包工具從這個暫存區製作套件。

## 案例一：Debian .deb（dpkg-deb）

### 方式 A：使用 debhelper（推薦）

`debian/rules` 僅需三行，`dh_auto_install` 自動辨識 Meson 並設好 `DESTDIR`[^debian]:

```makefile
#!/usr/bin/make -f
%:
	dh $@
```

`dh_auto_install` 內部執行 `DESTDIR=debian/tmp meson install -C builddir`，`dpkg-deb` 從 `debian/tmp` 讀取檔案。

### 方式 B：手動流程（無 dpkg-buildpackage）

```bash
meson setup builddir --prefix /usr
meson compile -C builddir
DESTDIR=/tmp/staging meson install -C builddir

mkdir -p /tmp/staging/DEBIAN
cat > /tmp/staging/DEBIAN/control <<EOF
Package: myapp
Version: 1.0-1
Section: devel
Priority: optional
Architecture: amd64
Maintainer: You <you@example.com>
Description: My Meson-built app
EOF

dpkg-deb --build /tmp/staging myapp_1.0-1_amd64.deb
```

## 案例二：RPM .rpm（rpmbuild）

### 方式 A：Fedora %meson 巨集（推薦）

Fedora 提供 `%meson`、`%meson_build`、`%meson_install` 巨集，內部自動處理 `DESTDIR=%{buildroot}`[^fedora]:

```spec
Name:    myapp
Version: 1.0
Release: 1%{?dist}
Summary: My app
License: MIT
BuildRequires: meson ninja-build

%build
%meson
%meson_build

%install
%meson_install

%files
%{_bindir}/*
```

### 方式 B：手動 RPM spec

```spec
%build
meson setup builddir --prefix %{_prefix} --buildtype=plain
meson compile -C builddir

%install
DESTDIR=%{buildroot} meson install -C builddir

%files
%{_bindir}/*
```

## 案例三：Flatpak（flatpak-builder）

Flatpak 不直接使用 `DESTDIR`；`flatpak-builder` 自動將 `--prefix=/app` 傳給 `meson setup`，安裝時檔案直接進沙箱內的 `/app`[^flatpak]:

```yaml
# com.example.MyApp.yml
id: com.example.MyApp
runtime: org.gnome.Platform
runtime-version: '48'
sdk: org.gnome.Sdk
command: my-app

finish-args:
  - --share=ipc
  - --socket=wayland

modules:
  - name: myapp
    buildsystem: meson
    config-opts:
      - --buildtype=release
    sources:
      - type: archive
        url: https://example.com/myapp-1.0.tar.gz
        sha256: abc123...
```

建置：

```bash
flatpak-builder --force-clean build-dir com.example.MyApp.yml
flatpak-builder --repo=myrepo --force-clean build-dir com.example.MyApp.yml
flatpak install --user myrepo com.example.MyApp
```

`buildsystem: meson` 等價於以下手動寫法：

```yaml
buildsystem: simple
build-commands:
  - meson setup builddir --prefix=/app --buildtype=release
  - ninja -C builddir install
```

## 模組輔助

Meson 提供以下相關模組支援打包生態系[^modules]:

| 模組 | 功能 | 狀態 |
|---|---|---|
| `import('pkgconfig')` | 產生 `.pc` 檔案，供 dpkg/rpm 的 **-dev** 套件使用 | 穩定 |
| `meson.add_install_script()` | 安裝時執行自訂腳本，可補充打包所需操作 | 穩定 |
| `meson.add_dist_script()` | 發行時執行腳本，可在原始碼封存前產生 packaging 檔案 | 穩定 |
| `import('rpm')` | 曾可產生 `.spec` 模板，但已於 2022 年初移除 | 已移除 |
| `import('debian')` | 從未存在 | — |

## 總結

| 目標格式 | 暫存機制 | 主要工具 | 自動化支援 |
|---|---|---|---|
| .deb | `DESTDIR=debian/pkg` + `meson install` | `dpkg-deb`、`dh_auto_install` | debhelper 自動偵測 Meson |
| .rpm | `DESTDIR=%{buildroot}` + `meson install` | `rpmbuild` | Fedora `%meson` / `%meson_install` 巨集 |
| Flatpak | 無需 DESTDIR，prefix=/app 直接安裝 | `flatpak-builder` | 內建 `buildsystem: meson` |
| 通用 | `DESTDIR=/staging` + `meson install` | 任意打包工具 | `add_install_script()` / `add_dist_script()` |

核心流程一致：`meson install` 搭配 `DESTDIR` 將檔案佈署至暫存區，再由各打包工具從該區讀取並封裝。

[^install]: Meson 團隊. (n.d.). Installing. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html
[^commands]: Meson 團隊. (n.d.). Command-line commands — `meson install`. Retrieved 2026-09-18, from https://mesonbuild.com/Commands.html
[^debian]: Debian 團隊. (n.d.). dh_auto_install manpage. Retrieved 2026-09-18, from https://manpages.debian.org/testing/debhelper/dh_auto_install.1.en.html
[^fedora]: Fedora 團隊. (n.d.). Fedora Packaging Guidelines — Meson. Retrieved 2026-09-18, from https://docs.fedoraproject.org/en-US/packaging-guidelines/Meson/
[^flatpak]: Flatpak 團隊. (n.d.). Flatpak Manifests. Retrieved 2026-09-18, from https://docs.flatpak.org/en/latest/manifests.html
[^modules]: Meson 團隊. (n.d.). Modules. Retrieved 2026-09-18, from https://mesonbuild.com/Modules.html