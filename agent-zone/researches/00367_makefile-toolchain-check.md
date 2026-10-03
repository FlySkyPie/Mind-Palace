# Makefile 中的工具鏈檢查：在執行任務前確認工具已安裝

## 概述

Makefile 在執行建置任務之前，可以透過多種方式檢查工具鏈是否完整。本報告整理了業界常見的檢查模式，包含工具是否存在、版本是否滿足最低要求、以及如何給出友善的錯誤訊息。

## 1. 基本檢查模式

### 1.1 `$(shell which ...)` — 解析階段檢查

在 Makefile 的頂層（任何 target 之前）使用 `$(shell ...)` 搭配 `ifeq`，可以在執行任何 target 之前就中斷並報錯[^gnu-make-shell]：

```makefile
PYTHON := $(shell which python3 2>/dev/null)
ifeq ($(PYTHON),)
$(error Python 3 是必需的但未找到。請安裝：apt install python3)
endif
```

這種方式會在執行任何 target 之前立即失敗，適合用於硬性要求。

### 1.2 `command -v` — 更可攜的替代方案

POSIX 標準的 `command -v` 比 `which` 在不同系統間的可攜性更好[^posix-command]：

```makefile
JQ := $(shell command -v jq 2>/dev/null)
ifeq ($(JQ),)
$(error jq 是必需的但未找到。請安裝：apt install jq)
endif
```

### 1.3 Recipe 內的延遲檢查

如果只在特定 target 需要某工具，可以在 recipe 內檢查[^makefile-tutorial]：

```makefile
deploy:
	@command -v kubectl >/dev/null 2>&1 || { echo "錯誤：需要 kubectl。請從 https://kubernetes.io/docs/tasks/tools/ 安裝"; exit 1; }
	kubectl apply -f deploy/
```

## 2. 友善的錯誤訊息

### 2.1 `$(error ...)` — 致命錯誤

最簡潔的方式在 Makefile 解析階段停止並給出訊息：

```makefile
ifeq ($(shell command -v cargo),)
$(error Rust/Cargo 是必需的但未安裝。請從 https://rustup.rs 安裝)
endif
```

### 2.2 最佳錯誤訊息應包含的三要素

Linux Kernel Makefile 示範了良好的錯誤訊息模式[^linux-kernel-makefile]：

```makefile
ifeq ($(filter output-sync,$(.FEATURES)),)
$(error GNU Make >= 4.0 是必需的。目前的 Make 版本是 $(MAKE_VERSION))
endif
```

好的錯誤訊息應包含：
- **什麼工具**缺失
- **你目前有什麼版本**（如果可取得）
- **如何安裝**或升級

### 2.3 `$(warning ...)` — 非致命警告

如果工具是選擇性的，使用警告而非錯誤：

```makefile
ifneq ($(shell command -v clang),)
CLANG_AVAILABLE := yes
else
$(warning 未找到 Clang；將跳過 Clang 專屬建置)
endif
```

## 3. 同時檢查多個工具

### 3.1 `REQUIRED_TOOLS` 變數搭配 `$(foreach)` 迴圈

```makefile
REQUIRED_TOOLS := git curl jq docker go
$(foreach tool,$(REQUIRED_TOOLS),\
    $(if $(shell command -v $(tool)),,\
        $(error 請安裝 $(tool)。Debian/Ubuntu 可用：sudo apt install $(tool))))
```

### 3.2 可重複使用的 `$(call REQUIRE ...)` 巨集

```makefile
# 巨集：REQUIRE(程式名稱, 套件名稱)
REQUIRE = \
    $(if $(shell command -v $(1)),,\
        $(error `$(1)` 是必需的但未找到。請安裝：apt install $(2)))

$(call REQUIRE, git,      git)
$(call REQUIRE, gcc,      build-essential)
$(call REQUIRE, python3,  python3)
$(call REQUIRE, node,     nodejs)
$(call REQUIRE, docker,   docker.io)
```

### 3.3 使用 `$(call ...)` 的「check-programs」函數

```makefile
check_program = \
    $(if $(shell command -v $(1)),,\
        $(error 找不到 $(1)。請安裝後再試。))

$(call check_program, git)
$(call check_program, cargo)
$(call check_program, npm)
```

### 3.4 專門的檢查 target

```makefile
REQUIRED_BINS := git:git cmake:cmake python3:python3

.PHONY: check-tools
check-tools:
	@$(foreach tool,$(REQUIRED_BINS),\
		bin=$(word 1,$(subst :, ,$(tool)));\
		pkg=$(word 2,$(subst :, ,$(tool)));\
		command -v $$bin >/dev/null 2>&1 || \
			(echo "錯誤：需要 $$bin。請安裝：apt install $$pkg"; exit 1);)
	@echo "所有必要工具都已就緒！"
```

## 4. 版本檢查

### 4.1 提取主要版本號進行數值比較

```makefile
# 檢查 GCC 版本 >= 10
GCC_VER := $(shell gcc -dumpfullversion 2>/dev/null || gcc -dumpversion 2>/dev/null)
GCC_VER_MAJOR := $(firstword $(subst ., ,$(GCC_VER)))
ifneq ($(GCC_VER),)
ifeq ($(shell [ $(GCC_VER_MAJOR) -ge 10 ] && echo yes),)
$(error GCC >= 10 是必需的。目前版本：$(GCC_VER)。請升級：apt install gcc-10)
endif
endif
```

### 4.2 使用 `sort -V` 進行語意化版本比較

```makefile
# 檢查 Node.js 版本 >= 18
NODE_VER := $(shell node --version 2>/dev/null | sed 's/v//')
MIN_NODE_VER := 18.0.0
ifneq ($(NODE_VER),)
ifeq ($(shell printf "$(MIN_NODE_VER)\n$(NODE_VER)\n" | sort -V | head -1),$(NODE_VER))
$(error Node.js >= 18.0.0 是必需的。目前版本：$(NODE_VER)。請使用 nvm 或從 https://nodejs.org 安裝)
endif
endif
```

### 4.3 Make 自身的版本檢查（Linux Kernel 風格）

Linux Kernel Makefile 使用功能特徵檢測而非版本字串比對[^linux-kernel-makefile]：

```makefile
ifeq ($(filter output-sync,$(.FEATURES)),)
$(error GNU Make >= 4.0 是必需的。目前的 Make 版本是 $(MAKE_VERSION))
endif
```

### 4.4 Python 版本檢查

```makefile
PYTHON := python3
PYTHON_VER := $(shell $(PYTHON) -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")' 2>/dev/null)
MIN_PYTHON := 3.10

ifneq ($(PYTHON_VER),)
ifeq ($(shell printf "%s\n%s\n" "$(MIN_PYTHON)" "$(PYTHON_VER)" | sort -V | head -1),$(PYTHON_VER))
$(error Python >= $(MIN_PYTHON) 是必需的。目前版本：$(PYTHON_VER)。請從 python.org 安裝)
endif
endif
```

### 4.5 Rust 工具鏈版本檢查

```makefile
RUSTC_VER := $(shell rustc --version 2>/dev/null | sed -E 's/^rustc ([0-9]+\.[0-9]+).*/\1/')
ifneq ($(RUSTC_VER),)
RUST_MIN := 1.70
ifeq ($(shell printf "%s\n%s\n" "$(RUST_MIN)" "$(RUSTC_VER)" | sort -t. -k1,1n -k2,2n | head -1),$(RUSTC_VER))
$(error Rust >= $(RUST_MIN) 是必需的。目前版本：$(RUSTC_VER)。請執行：rustup update)
endif
endif
```

## 5. 實戰案例

### 5.1 Linux Kernel Makefile

Linux Kernel 的 Makefile 在開頭就檢查 Make 版本是否夠新[^linux-kernel-makefile]：

```makefile
ifeq ($(filter output-sync,$(.FEATURES)),)
$(error GNU Make >= 4.0 is required. Your Make version is $(MAKE_VERSION))
endif
```

特點：
- 使用 `.FEATURES` 功能檢測而非脆弱的版本字串解析
- 同時報告需要的版本和目前的版本
- 在解析階段立即失敗，不執行任何建置步驟

### 5.2 Node.js Makefile

Node.js 的 Makefile 使用 `?=` 讓使用者可以從環境變數或命令列覆蓋工具路徑[^nodejs-makefile]：

```makefile
PYTHON ?= python3
EXEEXT := $(shell $(PYTHON) -c \
		"import sys; print('.exe' if sys.platform == 'win32' else '')")
```

### 5.3 Jacob Davis-Hansson 的「Sensible Defaults」

該 Makefile 在開頭檢查 `.RECIPEPREFIX` 功能是否存在[^jdh-make]：

```makefile
SHELL := bash
.ONESHELL:
.SHELLFLAGS := -eu -o pipefail -c
.DELETE_ON_ERROR:
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules

ifeq ($(origin .RECIPEPREFIX), undefined)
  $(error This Make does not support .RECIPEPREFIX. Please use GNU Make 4.0 or later)
endif
.RECIPEPREFIX = >
```

## 6. 完整的整合範例

以下是一個涵蓋所有常見需求的完整 Makefile 前綴範本：

```makefile
# ------------------------------------------------------------------
# 工具鏈驗證
# ------------------------------------------------------------------

# 巨集：檢查程式是否存在，失敗時給出友善訊息
define REQUIRE
$(if $(shell command -v $(1)),,\
    $(error `$(1)` 是必需的但未找到。\
        請安裝：apt install $(2)))
endef

# ----- 必要工具（失敗即中止）-----
$(call REQUIRE, git,       git)
$(call REQUIRE, gcc,       build-essential)
$(call REQUIRE, make,      build-essential)
$(call REQUIRE, cmake,     cmake)

# ----- 選擇性工具（僅警告）-----
HAVE_CLANG := $(shell command -v clang)
ifneq ($(HAVE_CLANG),)
CLANG_AVAILABLE := yes
$(info 找到選擇性工具：clang 在 $(HAVE_CLANG))
else
$(warning 未找到 clang；將跳過 Clang 專屬建置)
endif

# ----- 版本檢查 -----
# 檢查 Make 版本
ifeq ($(filter output-sync,$(.FEATURES)),)
$(error GNU Make >= 4.0 是必需的。目前版本：$(MAKE_VERSION))
endif

# 檢查 CMake 版本 >= 3.20
CMAKE_VER := $(shell cmake --version 2>/dev/null | head -1 | sed 's/[^0-9.]*//g')
CMAKE_MIN := 3.20
ifneq ($(CMAKE_VER),)
ifeq ($(shell printf "%s\n%s\n" "$(CMAKE_MIN)" "$(CMAKE_VER)" | sort -V | head -1),$(CMAKE_VER))
$(error CMake >= $(CMAKE_MIN) 是必需的。目前版本：$(CMAKE_VER))
endif
endif

# ----- 診斷 target -----
.PHONY: check-env
check-env:
	@echo "=== 工具鏈檢查 ==="
	@for tool in git gcc cmake make; do \
		path=$$(command -v "$$tool" 2>/dev/null); \
		if [ -n "$$path" ]; then \
			ver=$$($$tool --version 2>/dev/null | head -1); \
			echo "  ✓ $$tool: $$path ($$ver)"; \
		else \
			echo "  ✗ $$tool: 未找到"; \
		fi; \
	done

# ----- 建置 target -----
.PHONY: all
all: build

build:
	@echo "所有工具鏈檢查通過，準備建置！"
```

## 7. 最佳實踐總結

| 情境 | 建議模式 |
|---|---|
| 任何建置都需要某工具 | 在頂層使用 `$(call REQUIRE, tool, pkg)` 進行解析階段檢查 |
| 只在特定 target 需要某工具 | 在 recipe 內使用 `command -v` 搭配 `\|\| { echo "..."; exit 1; }` |
| 檢查最低版本 | 使用 `sort -V` 比較或功能檢測（如 `.FEATURES`） |
| 同時檢查多個工具 | `REQUIRED_TOOLS` 搭配 `$(foreach)` 迴圈 |
| 選擇性工具 | `HAVE_TOOL := $(shell command -v tool)` 搭配 `ifdef`/`ifndef` |
| 友善的錯誤訊息 | `$(error ...)` 應包含：缺失什麼、目前版本、如何安裝 |

## 參考文獻

[^gnu-make-shell]: Free Software Foundation. (n.d.). GNU Make Manual — The `shell` Function. Retrieved 2026-10-03, from https://www.gnu.org/software/make/manual/html_node/Shell-Function.html

[^posix-command]: IEEE/The Open Group. (n.d.). POSIX — `command` utility. Retrieved 2026-10-03, from https://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html

[^linux-kernel-makefile]: Linus Torvalds and the Linux Foundation. (n.d.). Linux Kernel Makefile. Retrieved 2026-10-03, from https://raw.githubusercontent.com/torvalds/linux/master/Makefile

[^jdh-make]: Jacob Davis-Hansson. (2021). Sensible Defaults for Make. Retrieved 2026-10-03, from https://tech.davis-hansson.com/p/make/

[^nodejs-makefile]: Node.js contributors. (n.d.). Node.js Makefile. Retrieved 2026-10-03, from https://raw.githubusercontent.com/nodejs/node/master/Makefile

[^makefile-tutorial]: The Makefile Tutorial. (n.d.). A Comprehensive Guide to Make. Retrieved 2026-10-03, from https://makefiletutorial.com/