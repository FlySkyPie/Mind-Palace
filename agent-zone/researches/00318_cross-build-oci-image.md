# OCI（Docker）映像檔跨平台建置：於 x86 機器上建置 ARM 映像

## 概述

本報告探討如何在 x86（amd64）機器上跨架構建置 OCI（Docker）映像檔，特別是 ARM64（aarch64）映像。主要方法包括 QEMU 模擬、Docker Buildx 多平台支援、Buildah 多架構模式，以及最佳效能的交叉編譯（cross-compilation）策略。

## 背景：為何需要跨平台建置？

開發者通常在 x86 筆電或 CI 上工作，但 ARM64（如 AWS Graviton、Apple Silicon、Raspberry Pi）已成主流部署目標。若無跨平台建置，團隊需另備 ARM 原生 CI runner 或於目標裝置上建置，增加複雜度與時間。

## 方法一：QEMU + binfmt_misc（基礎模擬層）

這是所有跨平台建置的底層機制。Linux 核心的 `binfmt_misc` 功能會檢查 ELF 檔案的 magic bytes，將非原生架構的二進位檔自動交由 QEMU user-mode 模擬器執行。[^qemu-binfmt]

### 安裝

單行指令即可註冊所有 QEMU binfmt handler：

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```

驗證：

```bash
ls -1 /proc/sys/fs/binfmt_misc/ | grep qemu
# 應看到: qemu-aarch64, qemu-arm, qemu-x86_64, ...
```

完成後即可在 x86 機器上執行 ARM64 容器：

```bash
docker run --rm --platform=linux/arm64 alpine uname -m
# 輸出: aarch64
```

> **注意**：Docker Desktop 已內建 QEMU 支援，無需額外設定。此處僅討論 Linux Engine 的作法。

## 方法二：Docker Buildx + `--platform`（多平台模擬建置）

Buildx（BuildKit 前端）是 Docker 官方推薦的多平台建置工具。[^buildx-multiarch]

### 建立 Builder 實例

```bash
docker buildx create \
  --name multiarch \
  --driver docker-container \
  --bootstrap
```

此步驟建立一個 BuildKit 容器來處理多平台建置。**預設的 `docker` driver 不支援多平台。**

### 建置多平台映像

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag yourname/image:tag \
  --push \
  .
```

### 關鍵限制：`--push` 與 `--load`

| 選項 | 多平台行為 | 用途 |
|---|---|---|
| `--push` | ✅ 將所有平台映像上傳至 registry，並寫入 OCI manifest | 正式使用 |
| `--load` | ❌ 僅建置宿主原生架構，其餘架構被忽略 | 僅適用單一平台 |

### 完整 Dockerfile 範例

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:latest
RUN uname -m > /arch
CMD cat /arch
```

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t multi-test --push .
```

在 x86 機器上執行 `docker run multi-test` 輸出 `x86_64`，在 ARM 機器上輸出 `aarch64`。

## 方法三：交叉編譯 + 多階段建置（最佳效能）

這是**最推薦的方法**——避免整個 toolchain 在 QEMU 模擬下執行，而是將**建置階段釘選在宿主原生架構**，僅對編譯器下達交叉編譯參數。[^docker-cross-compile]

### 核心模式

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM golang:alpine AS build
ARG TARGETOS
ARG TARGETARCH
WORKDIR /src
COPY . .
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o /out/server .

FROM alpine
COPY --from=build /out/server /server
```

### BuildKit 自動注入的 ARG

| 變數 | 範例值 | 說明 |
|---|---|---|
| `BUILDPLATFORM` | `linux/amd64` | 建置機的架構 |
| `BUILDOS` | `linux` | BUILDPLATFORM 的 OS 部分 |
| `BUILDARCH` | `amd64` | BUILDPLATFORM 的架構部分 |
| `TARGETPLATFORM` | `linux/arm64` | `--platform` 指定的目標 |
| `TARGETOS` | `linux` | TARGETPLATFORM 的 OS 部分 |
| `TARGETARCH` | `arm64` | TARGETPLATFORM 的架構部分 |

### 為什麼更快？

- 若不使用 `--platform=$BUILDPLATFORM`，**每個 `RUN` 指令**（包括 `apt install`、`go build`）都在 QEMU 模擬下執行，通常比原生慢 3–5 倍。
- 使用後，編譯器以原生速度執行，僅最終執行階段（通常只是 `COPY --from-build`）需要模擬。
- Docker 官方基準測試（Go 專案）：全 QEMU 模擬約 280 秒，交叉編譯約 50 秒，**提速約 5.6 倍**。

### 各語言範例

**Go：**

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.22-alpine AS build
ARG TARGETOS TARGETARCH
WORKDIR /src
COPY go.mod go.sum .
RUN go mod download
COPY . .
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o /out/server .
```

**Rust（使用 `xx-cargo`）：**

```dockerfile
FROM --platform=$BUILDPLATFORM tonistiigi/xx AS xx
FROM --platform=$BUILDPLATFORM rust:alpine
COPY --from=xx / /
ARG TARGETPLATFORM
RUN xx-cargo build --release --target-dir ./build
```

**Python / Node.js（純直譯）：**

純 Python 或純 JS 不需交叉編譯，但 numpy、bcrypt 等 C extensions 需要。建議直接使用目標架構基底：

```dockerfile
FROM --platform=$TARGETPLATFORM node:22-alpine
# 或
FROM --platform=$TARGETPLATFORM python:3.12-slim
```

## 方法四：`tonistiigi/xx` 交叉編譯輔助工具

`xx` 是一組 shell 腳本（以 Docker 映像發佈），簡化 Dockerfile 中的交叉編譯設定。[^tonistiigi-xx]

### 使用方式

```dockerfile
FROM --platform=$BUILDPLATFORM tonistiigi/xx AS xx
FROM --platform=$BUILDPLATFORM your-base-image
COPY --from=xx / /
ARG TARGETPLATFORM
```

### 常用指令

| 指令 | 用途 |
|---|---|
| `xx-info env` | 印出所有目標變數 |
| `xx-info triple` | 印出 GCC 風格的 target triple |
| `xx-apk add` | 安裝 Alpine 目標架構套件 |
| `xx-apt-get install` | 安裝 Debian 目標架構套件 |
| `xx-go` | 交叉編譯 Go（自動設定 `GOOS`/`GOARCH`） |
| `xx-cargo` | 交叉編譯 Rust（自動設定 target triple） |
| `xx-verify` | 驗證二進位檔是否為目標架構 |

### 完整 Go 範例

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$BUILDPLATFORM tonistiigi/xx AS xx
FROM --platform=$BUILDPLATFORM golang:1.22-alpine AS build
COPY --from=xx / /
ARG TARGETPLATFORM
ENV CGO_ENABLED=0
WORKDIR /src
COPY . .
RUN xx-go build -o /out/server . && xx-verify /out/server

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=build /out/server /server
```

## 方法五：Buildah 多架構模式

Buildah（Podman/Containers 生態系）支援 `--arch` 與 `--platform` 旗標。[^buildah-multiarch]

### 安裝

```bash
sudo apt install -y podman buildah qemu-user-static
sudo podman run --rm --privileged docker.io/multiarch/qemu-user-static --reset -p yes
```

### 使用 Manifest 方式

```bash
buildah manifest create myapp-multiarch
buildah build --arch amd64 --tag "ghcr.io/user/myapp:v1.0.0" --manifest myapp-multiarch .
buildah build --arch arm64 --tag "ghcr.io/user/myapp:v1.0.0" --manifest myapp-multiarch .
buildah manifest push --all myapp-multiarch "docker://ghcr.io/user/myapp:v1.0.0"
```

Buildah 的優點是 daemonless 且 rootless，不需背景 daemon。

## 方法六：原生多節點叢集（最高效能）

不依賴模擬，而是將不同架構的真實機器加入單一 Buildx builder 叢集：[^buildx-cluster]

```bash
# 加入本機 x86 節點
docker buildx create --name mybuild --use unix:///var/run/docker.sock

# 加入遠端 ARM64 節點（透過 SSH）
docker buildx create --append --name mybuild ssh://user@arm64-host

# 跨兩種原生節點建置
docker buildx build --platform linux/amd64,linux/arm64 --push .
```

## 注意事項與陷阱

### `RUN` 指令的二進位執行

- **套件管理員**（`apt-get`、`apk add`）在 QEMU 下可執行但極慢。
- **pip install / npm install** 若有原生 C extensions（numpy、bcrypt），在 QEMU 下慢 10–20 倍。
- **Go/Rust 建置**應避免在 QEMU 下執行，務必使用交叉編譯。

### 已知 QEMU 限制

- 某些 CPU 特定指令（如 Go 加密 assembly 最佳化）可能使 QEMU 崩潰。解決：`export CGO_ENABLED=0`。
- QEMU user-mode **不支援** `clone()` 搭配 `CLONE_NEWNS`（無法建立新 mount namespace）。
- ARM32（arm/v7）在 ARM64 宿主上的 QEMU 模擬特別不穩定。

### 快取行為

- **快取掛載（`--mount=type=cache`）是每個架構獨立的**，不共享。
- 在 CI 中務必設定 `cache-from`/`cache-to`。GitHub Actions 使用 `type=gha`，其他 CI 使用 `type=registry`。

### 基底映像注意事項

- **優先使用多架構基底映像**如 `alpine`、`ubuntu`、`node`、`python`、`golang`、`rust`，這些映像附帶 manifest list 支援所有架構。
- 若使用單一架構基底（如 `arm64v8/python`），其他平台將無法正確解析。

## 效能比較

| 方法 | 速度 | 複雜度 | 適用場景 |
|---|---|---|---|
| QEMU 模擬（僅） | 比原生慢 3–5 倍 | 低（無需修改 Dockerfile） | 快速測試、簡單腳本 |
| 交叉編譯（多階段 + `$BUILDPLATFORM`） | 約原生 1.7 倍（2 平台） | 中（需修改 Dockerfile） | 編譯型語言（Go, Rust, C++） |
| 原生多節點（叢集） | 每平台原生速度 | 高（需基礎架構） | 正式 CI/CD |
| Docker Build Cloud | 每平台原生速度 | 低（託管服務） | 團隊無基礎架構需求 |

實際測試（Docker CLI 官方專案）：單平台原生 ~50 秒，最佳化交叉編譯（2 平台）~85 秒，全 QEMU 模擬 ~280 秒。

## 快速決策流程

```
是否能修改 Dockerfile？
├─ 否 → 使用 QEMU 模擬（tonistiigi/binfmt + buildx --platform）
└─ 是 → 語言是否支援交叉編譯？
         ├─ 是（Go, Rust, C/C++, Zig）→ 多階段 + $BUILDPLATFORM + xx
         └─ 否（Python C-ext, Node 原生模組）→ QEMU 模擬或原生 ARM runner
```

## 結論

在 x86 機器上建置 ARM64 映像的主要方法有：

1. **QEMU + binfmt_misc** 提供底層執行能力
2. **Docker Buildx `--platform`** 是最簡單的 Docker 原生方式
3. **交叉編譯多階段建置**是編譯型語言的最佳效能方案
4. **`tonistiigi/xx`** 大幅簡化交叉編譯腳本
5. **Buildah** 提供 daemonless 的替代方案
6. **原生多節點叢集**提供最高效能但需要額外硬體

對於正式專案，建議優先採用 **交叉編譯 + `$BUILDPLATFORM` 多階段建置**策略，兼顧速度與架構支援。

---

[^qemu-binfmt]: Docker Blog. (2025). Multi-platform builds with Docker Buildx and QEMU. Retrieved 2026-10-01, from https://docs.docker.com/build/building/multi-platform/

[^buildx-multiarch]: Docker Inc. (n.d.). Multi-platform builds. Retrieved 2026-10-01, from https://docs.docker.com/build/building/multi-platform/

[^docker-cross-compile]: Docker Inc. (2024). Faster multi-platform builds — Dockerfile cross-compilation guide. Retrieved 2026-10-01, from https://www.docker.com/blog/faster-multi-platform-builds-dockerfile-cross-compilation-guide/

[^tonistiigi-xx]: Tonistiigi. (n.d.). xx — Cross compilation helper. Retrieved 2026-10-01, from https://github.com/tonistiigi/xx

[^buildah-multiarch]: Buildah Project. (2023). Multi-architecture builds with Buildah. Retrieved 2026-10-01, from https://feldspaten.org/2023/06/01/Multi-Arch-Buildah/

[^buildx-cluster]: Docker Inc. (n.d.). Setting up a remote builder with Buildx. Retrieved 2026-10-01, from https://docs.docker.com/build/buildx/install/