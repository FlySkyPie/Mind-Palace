# BitBake 空專案建置與 C++ 建置目標整合指南

## 概要

本指南說明如何從頭建立一個 BitBake (OpenEmbedded/Yocto Project) 的空專案，並將自訂的 C++ 建置目標整合進來。BitBake 是 OpenEmbedded 建置系統的核心工具，負責解析中繼資料、產生任務清單、排程編譯[^bitbake-concept]。

## 環境建置方案

Poky 的舊有單一倉庫（`https://git.yoctoproject.org/poky/`）已不再維護 master 分支。官方提供兩種推薦方案[^poky-deprecated]：

### 方案 A（推薦）：使用 bitbake-setup

```bash
python3 -m venv --clear ./bitbake-setup-venv
. ./bitbake-setup-venv/bin/activate
pip install bitbake-setup
bitbake-setup init
```

互動式選擇 Configuration Template 後，會自動建立以下目錄結構[^bitbake-setup]：

```
./bitbake-builds/
├── site.conf
└── <setup-name>/
    ├── build/
    ├── config/
    └── layers/
```

接著 Source 建置環境：

```bash
source ./<setup-name>/build/init-build-env
```

### 方案 B：手動克隆個別 Layer

```bash
mkdir bitbake-builds && cd bitbake-builds
mkdir layers/
git clone -b yocto-6.0.3 https://git.openembedded.org/bitbake ./layers/bitbake
git clone -b yocto-6.0.3 https://git.openembedded.org/openembedded-core ./layers/openembedded-core
git clone -b yocto-6.0.3 https://git.yoctoproject.org/meta-yocto ./layers/meta-yocto
source ./layers/openembedded-core/oe-init-build-env
```

此命令會：
- 建立 `build/` 目錄
- 產生 `build/conf/local.conf` 與 `build/conf/bblayers.conf`
- 設定必要的環境變數（`PATH`、`BBPATH`、`BUILDDIR`）
- 讓 `bitbake` 指令可在當前 shell 中使用[^oe-init-env]

## 建立自訂 Layer

Layer 是 BitBake 中組織中繼資料的機制。以下步驟建立一個名為 `meta-mylayer` 的自訂 layer：

### 目錄結構

```
meta-mylayer/
├── conf/
│   └── layer.conf
├── recipes-example/
│   └── hello-cmake/
│       ├── files/
│       │   ├── CMakeLists.txt
│       │   └── main.cpp
│       └── hello-cmake.bb
├── COPYING.MIT
└── README
```

### layer.conf

在 `meta-mylayer/conf/layer.conf` 中定義 layer 的基本資訊：

```bitbake
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "mylayer"
BBFILE_PATTERN_mylayer = "^${LAYERDIR}/"
BBFILE_PRIORITY_mylayer = "6"
```

關鍵變數說明：
- `BBFILE_COLLECTIONS`：註冊 layer 名稱
- `BBFILE_PATTERN_mylayer`：比對此 layer 中的 recipe 檔案路徑
- `BBFILE_PRIORITY_mylayer`：當多個 layer 提供相同 recipe 時的優先順序，數字愈大優先權愈高[^layer-conf]

### 註冊 Layer 至 bblayers.conf

編輯 `build/conf/bblayers.conf`，將 `meta-mylayer` 加入 `BBLAYERS`：

```bitbake
BBLAYERS ?= " \
  /path/to/poky/meta \
  /path/to/poky/meta-poky \
  /path/to/poky/meta-yocto-bsp \
  /path/to/meta-mylayer \
  "
```

也可使用 `bitbake-layers add-layer` 指令自動加入（會產生絕對路徑）[^bitbake-layers-add]。

## 撰寫 C++ Recipe（CMake 版）

### 原始碼準備

**`hello-cmake/files/CMakeLists.txt`**：

```cmake
cmake_minimum_required(VERSION 3.10)
project(hello-cmake LANGUAGES CXX)
add_executable(hello-cmake main.cpp)
install(TARGETS hello-cmake RUNTIME DESTINATION bin)
```

**`hello-cmake/files/main.cpp`**：

```cpp
#include <iostream>
int main() {
    std::cout << "Hello from BitBake!" << std::endl;
    return 0;
}
```

### Recipe 內容

建立 `hello-cmake/hello-cmake.bb`：

```bitbake
SUMMARY = "Simple C++ application built with CMake"
DESCRIPTION = "A hello world example demonstrating CMake-based C++ build in BitBake"
SECTION = "examples"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = " \
    file://CMakeLists.txt \
    file://main.cpp \
"

S = "${WORKDIR}"

inherit cmake

EXTRA_OECMAKE = ""
```

關鍵說明：
- `inherit cmake`：匯入 BitBake 內建的 CMake 類別，它會自動處理 configure、compile、install 階段
- `S = "${WORKDIR}"`：原始碼目錄指向工作目錄（因為檔案來自 SRC_URI 而非遠端 tarball）
- `EXTRA_OECMAKE`：可傳遞額外的 CMake 參數（如 `-DCMAKE_BUILD_TYPE=Release`）[^cmake-recipe]

## 進階 Recipe 變體

### 從 Git 倉庫擷取原始碼

若 C++ 專案存放在 Git 倉庫：

```bitbake
SRC_URI = "git://github.com/username/my-cpp-project.git;branch=main;protocol=https"
SRCREV = "abc123def456..."
S = "${WORKDIR}/git"

inherit cmake
```

使用 `git://` 協定與 `SRCREV` 指定特定 commit 來確保可重複建置[^git-fetch]。

### 非 CMake 的 C++ Recipe（使用 Makefile）

```bitbake
SUMMARY = "Simple C++ with Makefile"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://main.cpp file://Makefile"
S = "${WORKDIR}"

do_compile() {
    ${CC} ${CFLAGS} main.cpp -o hello-cpp ${LDFLAGS}
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello-cpp ${D}${bindir}
}
```

此方式直接使用 `do_compile` 與 `do_install` 手動定義建置步驟，`${CC}`、`${CFLAGS}`、`${LDFLAGS}` 由 BitBake 自動設定[^manual-recipe]。

## 將 Recipe 納入 Image

### 加入 local.conf

編輯 `build/conf/local.conf`，加入：

```bitbake
IMAGE_INSTALL:append = " hello-cmake"
```

`IMAGE_INSTALL` 控制最終 rootfs 中包含哪些套件。`[:append]` 語法附加而不覆蓋既有值[^image-install]。

### 驗證

```bash
bitbake hello-cmake      # 只建置此 recipe
bitbake core-image-minimal  # 建置完整 image（含 hello-cmake）
```

使用 `bitbake -g hello-cmake` 可產生建置依賴圖[^bitbake-build]。

## 常見問題

### 找不到 layer

確認 `bblayers.conf` 中的路徑為絕對路徑且指向正確目錄，並檢查 `layer.conf` 的 `BBFILE_COLLECTIONS` 是否正確註冊。

### Recipe 未被解析

確保 `.bb` 檔案放置在 `recipes-*/<package>/` 目錄結構中，並確認 layer 路徑已加入 `BBLAYERS`。

### CMake 找不到 toolchain

`inherit cmake` 會自動載入 Yocto 的 CMake toolchain file，通常無需手動設定。

## 結論

建立 BitBake 空專案並整合 C++ 建置目標的主要步驟歸納為：
1. 取得 OpenEmbedded 並初始化建置環境（bitbake-setup 推薦，或手動克隆個別 layer）
2. 建立自訂 meta-layer（含 `conf/layer.conf`）
3. 撰寫 recipe（`inherit cmake` 最簡潔，或手動定義 `do_compile`）
4. 將 layer 加入 `bblayers.conf`，將 recipe 加入 `IMAGE_INSTALL`
5. 執行 `bitbake` 建置目標

[^poky-deprecated]: Yocto Project. (n.d.). _Setting Up the Poky Reference Distro Manually_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/dev-manual/poky-manual-setup.html
[^bitbake-setup]: Yocto Project. (n.d.). _Setting Up The Environment With bitbake-setup_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/bitbake-user-manual/bitbake-user-manual-environment-setup.html
[^bitbake-concept]: Yocto Project. (n.d.). _BitBake Concepts_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/overview-manual/concepts.html
[^oe-init-env]: Yocto Project. (n.d.). _Building - oe-init-build-env_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/dev-manual/building.html
[^layer-conf]: Yocto Project. (n.d.). _Creating Your Own Layer_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/dev-manual/dev-manual.html#creating-your-own-layer
[^bitbake-layers-add]: Toradex. (2026-07-16). _Custom meta layers, recipes and images in Yocto Project (hello world examples)_. Retrieved 2026-09-13, from https://developer.toradex.com/linux-bsp/os-development/build-yocto/custom-meta-layers-recipes-and-images-in-yocto-project-hello-world-examples/
[^cmake-recipe]: joaocfernandes. (n.d.). _Learn-Yocto / Recipe-CMake.md_. Retrieved 2026-09-13, from https://github.com/joaocfernandes/Learn-Yocto/blob/master/develop/Recipe-CMake.md
[^git-fetch]: Yocto Project. (n.d.). _BitBake Fetching - Git_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/dev/bitbake-user-manual/bitbake-user-manual-fetching.html
[^manual-recipe]: wolfSSL. (n.d.). _Beginners Guide to Writing a Recipe for OpenEmbedded and Yocto Projects_. Retrieved 2026-09-13, from https://www.wolfssl.com/docs/yocto-openembedded-recipe-guide/
[^image-install]: Yocto Project. (n.d.). _Variables Glossary - IMAGE_INSTALL_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/ref-manual/variables.html#term-IMAGE_INSTALL
[^bitbake-build]: Yocto Project. (n.d.). _BitBake User Manual - Hello World Example_. Retrieved 2026-09-13, from https://docs.yoctoproject.org/bitbake/dev/bitbake-user-manual/bitbake-user-manual-hello.html