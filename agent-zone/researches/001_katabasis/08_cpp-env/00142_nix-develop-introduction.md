# Nix Develop 介紹：現代 Nix 開發環境的基礎

## 什麼是 `nix develop`？

`nix develop` 是 Nix 套件管理器中基於 flakes 的指令，用於啟動一個**與 Nix 建置環境幾乎完全相同**的互動式開發 shell。它屬於實驗性的新 CLI（new CLI）的一部分，需要啟用 `nix-command` 和 `flakes` 兩個實驗性功能。

根據官方手冊的定義[^nix-develop-manual]：

> `nix develop` 啟動一個 bash shell，提供與 Nix 用於建置 installable 時幾乎相同的互動式建置環境。在這個 shell 中，環境變數和 shell 函數都已設定完畢，讓你可以互動且增量地建置你的套件。

它解決的核心問題是傳統開發環境中的「在我機器上可以跑（works on my machine）」症候群——不同開發者安裝了不同版本的編譯器、函式庫和工具，導致環境不一致的問題。

## `nix develop` vs. `nix-shell`（經典指令）

`nix develop` 並非從零發明的新概念，而是 `nix-shell` 在 flakes 生態系中的現代取代品。以下是兩者的詳細比較：

| 面向 | `nix develop`（新 CLI） | `nix-shell`（經典） |
|------|------------------------|---------------------|
| **狀態** | 實驗性（experimental） | 穩定（stable） |
| **專案類型** | flake 專案（`flake.nix`） | 非 flake 專案（`shell.nix` / `default.nix`） |
| **設定檔** | `flake.nix` → `devShells.<system>.default` | `shell.nix` → `default.nix` |
| **可重現性** | 內建 via `flake.lock` | 需手動 via `-I` 鎖定 nixpkgs |
| **建置階段** | 內建 flags：`--build`, `--configure` 等 | 手動 `eval ${buildPhase:-buildPhase}` |
| **臨時套件** | 使用 `nix shell nixpkgs#<pkg>`（注意是 `nix shell` 非 `nix develop`） | `nix-shell -p <pkgs>` |
| **純淨模式** | `--ignore-env` / `-i` | `--pure` |

NixOS Wiki 直接指出[^nixos-wiki-devenv]：

> 對於 flakes 為基礎的專案（專案根目錄有 `flake.nix`），我們用 `nix develop` 取代 `nix-shell`。

### 重要的名稱釐清

官方手冊特別說明一個常見混淆點[^nix3-env-shell]：

- **`nix-shell`**（經典，穩定）——上述的傳統指令
- **`nix shell`** / **`nix env shell`**（新 CLI，實驗性）——這是一個**不同**的新指令，用於在 `$PATH` 中加入指定套件並啟動 shell，語法如 `nix shell nixpkgs#hello`
- **`nix develop`**（新 CLI，實驗性）——提供完整的**建置環境**（包含環境變數、建置階段函數等），不只是把套件放到路徑上

## 如何開始使用

### 最基本範例：`flake.nix`（推薦）

```nix
{
  description = "一個基本的 flake 含 dev shell";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";
  };

  outputs = { self, nixpkgs }:
    let
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
    in
    {
      devShells.x86_64-linux.default = pkgs.mkShell {
        packages = with pkgs; [
          cowsay
          lolcat
        ];

        shellHook = ''
          echo "歡迎來到 dev shell！"
        '';
      };
    };
}
```

使用方式：

```bash
$ nix develop
```

### 指定名稱的 devShell

```nix
devShells.x86_64-linux.myproject = pkgs.mkShell { ... };
```

```bash
$ nix develop .#myproject
```

### 傳統 `shell.nix` 方式（經典 `nix-shell`）

若專案尚未採用 flakes，仍可使用 `shell.nix` 搭配 `nix-shell`[^nix-dev-declarative]：

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-24.05";
  pkgs = import nixpkgs { config = {}; overlays = []; };
in
pkgs.mkShellNoCC {
  packages = with pkgs; [ cowsay lolcat ];
  GREETING = "Hello, Nix!";
  shellHook = ''
    echo $GREETING | cowsay | lolcat
  '';
}
```

### `nix develop` 常用選項

```bash
nix develop                          # 使用 devShells.<system>.default 或 packages.<system>.default
nix develop nixpkgs#hello            # 以 nixpkgs 中的 hello 套件的建置環境
nix develop .#myapp                  # 指定名稱的 devShell
nix develop --command bash -c "make" # 執行單一指令而非啟動互動 shell
nix develop --build                  # 直接執行 buildPhase
nix develop --configure              # 直接執行 configurePhase
nix develop --profile /tmp/my-env     # 記錄建置環境至 profile
```

## 與其他開發環境工具比較

### `nix develop` vs. `direnv`

這兩者並非競爭關係，而是**互補工具**[^nix-dev-direnv]：

- `nix develop` 定義了開發環境中有哪些工具可用
- `direnv` + `nix-direnv` 讓你在 `cd` 進入專案目錄時**自動啟用** Nix shell

```bash
$ echo "use flake" > .envrc && direnv allow
```

從此不需要手動輸入 `nix develop`，進出目錄時環境會自動載入和卸載。

### `nix develop` vs. Devbox

Devbox（由 Jetify 開發）是構建在 **Nix 之上的高層工具**[^devbox]：

| 面向 | `nix develop` | Devbox |
|------|--------------|--------|
| **設定格式** | Nix 語言（`flake.nix`） | JSON（`devbox.json`） |
| **學習曲線** | 陡峭（需學習 Nix 語言） | 較低（JSON 較簡單） |
| **底層引擎** | Nix 直接 | Nix + 自訂抽象層 |
| **靈活性** | 極高 | 中等 |
| **CLI 簡潔度** | `nix develop` | `devbox shell` |

Devbox 本質上是 Nix 的友好封裝，適合不想深入學習 Nix 的團隊。

### `nix develop` vs. Docker / VirtualEnv / asdf

- **Docker**：提供完整的 OS 層級隔離，但較重。`nix develop` 更輕量（僅設定環境變數和 `PATH`），無需容器或 daemon。
- **VirtualEnv / pyenv**：語言特定。`nix develop` 從單一定义管理所有語言和工具的整個 toolchain。
- **asdf / mise**：版本管理器，處理個別工具。`nix develop` 整體性地管理整個 toolchain，具備真正的可重現性。

## 優缺點分析

### 優點 ✅

- **可重現性**：透過 `flake.lock` 鎖定依賴，無論本地開發、CI 或團隊成員，環境完全一致
- **隔離性**：工具僅在 shell 內可用，不汙染全域系統
- **多語言支援**：從單一定义管理 compiler、interpreter、函式庫和系統工具
- **增量建置**：環境與實際 Nix 建置幾乎相同，可逐階段除錯（`configurePhase`、`buildPhase` 等）
- **快取機制**：Nix 全域快取建置輸出，初次建置後即可快速載入
- **Git 友好**：`flake.nix` 是純文字檔，可納入版本控制與 Code Review

### 缺點 ❌

- **實驗性**：仍標記為實驗性功能，介面可能變動
- **僅限 flake**：不支援 `nix-shell -p` 風格的臨時套件啟用（這部分需用 `nix shell` 替代）
- **學習曲線陡峭**：需理解 Nix flakes、Nix 語言和 `mkShell`，對新手門檻較高
- **無自動啟用**：不像 `direnv` 自動載入，每次需手動執行 `nix develop`（或搭配 `direnv` 使用）
- **Nix 依賴**：使用者需安裝 Nix 並啟用 flakes
- **啟動延遲**：首次評估可能較慢（後續會快取）
- **僅支援 bash**：預設僅使用 bash（可透過 `NIX_BUILD_SHELL` 覆蓋）

## 實際應用場景

### 1. 開發 Nix 本身

NixOS Wiki 展示了 Nix 專案如何使用 `nix develop` 進行增量開發[^nixos-wiki-devenv]：

```bash
$ git clone https://github.com/NixOS/nix --depth 1
$ cd nix
$ nix develop
$ ./bootstrap.sh
$ ./configure $configureFlags --prefix=$(pwd)/outputs/out
$ make -j $NIX_BUILD_CORES
```

Shell 內已設定好所有建置依賴（autoconf、boost 等），開發者可快速迭代。

### 2. 多語言開發環境

一個 Python + Node.js + Rust 的開發環境：

```nix
devShells.x86_64-linux.default = pkgs.mkShell {
  packages = with pkgs; [
    python311
    nodejs_20
    cargo
    rustc
    pkg-config
    openssl
    gcc
  ];
  shellHook = ''
    echo "開發環境已載入：Python, Node.js, Rust"
  '';
};
```

### 3. 搭配 direnv 自動啟用

```bash
$ echo "use flake" > .envrc
$ direnv allow
```

從此 `cd` 進專案目錄時自動啟用 Nix dev shell，離開時自動卸載。

### 4. CI/CD 可重現性

定義單一 `flake.nix`，本地開發者與 CI（GitHub Actions、GitLab CI）共用。CI 可執行：

```yaml
- name: Build project
  run: nix develop --command make build
```

這保證了所有環境使用完全相同的工具版本進行建置。

## 總結

`nix develop` 是 Nix 生態系中現代化開發環境管理的核心指令，透過 flakes 提供可重現、隔離且宣言式的開發環境。它取代了傳統的 `nix-shell`，並可與 `direnv` 等工具完美搭配，形成強大的開發工作流程。

[^nix-develop-manual]: NixOS. (n.d.). nix develop - run a bash shell that provides the build environment of a derivation. *Nix 2.34.9 Reference Manual*. Retrieved 2026-09-25, from https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-develop.html

[^nixos-wiki-devenv]: NixOS Wiki. (n.d.). Development environment with nix-shell. Retrieved 2026-09-25, from https://wiki.nixos.org/wiki/Development_environment_with_nix-shell

[^nix3-env-shell]: NixOS. (n.d.). nix shell / nix env shell — run a shell in which the specified packages are available. *Nix 2.34.9 Reference Manual*. Retrieved 2026-09-25, from https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-env-shell.html

[^nix-dev-declarative]: nix.dev. (n.d.). First steps: Declarative shell. Retrieved 2026-09-25, from https://nix.dev/tutorials/first-steps/declarative-shell

[^nix-dev-direnv]: nix.dev. (n.d.). Recipes: direnv. Retrieved 2026-09-25, from https://nix.dev/guides/recipes/direnv

[^devbox]: Jetify. (n.d.). Devbox Quickstart. Retrieved 2026-09-25, from https://www.jetify.com/devbox/docs/quickstart/