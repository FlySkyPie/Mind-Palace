# Cuberite 插件開發指南：以 C/C++ 動態函式庫（DLL/.so）與 Lua 繫結

## 前言

Cuberite 是一款高效能的 Minecraft 伺服器軟體，其核心以 C++ 撰寫。原生的插件系統僅支援 **Lua 指令碼**——所有官方與第三方插件均以 Lua 撰寫，透過 tolua++ 自動生成繫結層呼叫 Cuberite 的 C++ API[^api-tutorial]。

然而，若需要在插件中執行高效能運算、呼叫底層系統 API（如 GPIO、硬體加速、自訂通訊協定）、或重複利用既有 C/C++ 函式庫，開發者仍可透過 **Lua C Module** 機制，將 C/C++ 動態函式庫（`.so` / `.dll`）載入 Cuberite 插件中使用。這份指南將詳細說明三種程度的實現方式。

[^api-tutorial]: Cuberite Contributors. (n.d.). Writing a Cuberite plugin. Retrieved 2026-09-13, from https://api.cuberite.org/Writing-a-Cuberite-plugin.html

---

## 目錄

1. [背景：Cuberite 的 Lua 與 C/C++ 架構](#1-背景cuberite-的-lua-與-cc-架構)
2. [方式一：Lua C Module（最成熟、推薦）](#2-方式一lua-c-module最成熟推薦)
3. [方式二：針對核心開發者的 tolua++ 繫結](#3-方式二針對核心開發者的-tolua-繫結)
4. [方式三：未來展望——原生 C/C++ 插件系統](#4-方式三未來展望原生-cc-插件系統)
5. [比較與建議](#5-比較與建議)
6. [參考資料](#6-參考資料)

---

## 1. 背景：Cuberite 的 Lua 與 C/C++ 架構

### 1.1 Cuberite 使用標準 Lua 5.1，非 LuaJIT

Cuberite 內嵌 **PUC-Rio Lua 5.1**（非 LuaJIT），因此 **LuaJIT FFI**（`ffi.load()`、`ffi.cdef()`）無法使用[^lua-version]。所有 C/C++ 與 Lua 之間的互動須透過 Lua 5.1 的 **C API**（`lua_State` 堆疊模型）進行。

[^lua-version]: Cuberite Contributors. (n.d.). Cuberite User's Manual - Plugins. Retrieved 2026-09-13, from https://book.cuberite.org/#2-5-plugins

### 1.2 `-rdynamic` 編譯旗標

Cuberite 在 Linux 上以 `-rdynamic` 旗標編譯，這使得主執行檔中所有符號（包含 Lua C API 函式如 `lua_open`、`lua_pushstring` 等）在動態載入的 `.so` 中均可解析[^rdynamic]。這是外部 C 模組得以順利運作的關鍵。

[^rdynamic]: Cuberite Contributors. (n.d.). src/CMakeLists.txt. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/CMakeLists.txt

### 1.3 `package.cpath` 自動設定

Cuberite 的 `cPluginLua` 在載入每個插件時，會自動將該插件的資料夾加入 `package.cpath`[^cpath-setup]：

```lua
-- 實際效果等同於：
package.cpath = package.cpath .. ";/path/to/plugin/?.so"
-- Windows 上為：
package.cpath = package.cpath .. ";\\path\\to\\plugin\\?.dll"
```

因此插件可直接以 `require("模組名稱")` 載入同一資料夾下的 `.so` / `.dll`。

[^cpath-setup]: Cuberite Contributors. (n.d.). src/Bindings/PluginLua.cpp (lines 96-102). Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/Bindings/PluginLua.cpp

### 1.4 Cuberite 的繫結系統概覽

Cuberite 的 Lua API 繫結由三層構成[^export-api]：

| 層級 | 工具 | 用途 | 誰可使用 |
|---|---|---|---|
| 自動繫結 | tolua++ | 從 C++ 標頭自動生成繫結程式碼 | 僅限 Cuberite 核心開發者 |
| 手動繫結 | ManualBindings*.cpp | 處理複雜型別（回呼、UUID、陣列） | 僅限 Cuberite 核心開發者 |
| Lua C Module | `require()` + `luaopen_*` | 載入外部 C 共享函式庫 | **插件開發者可用** |

[^export-api]: Cuberite Contributors. (n.d.). Exporting Cuberite API to Lua. Retrieved 2026-09-13, from http://cuberite.xoft.cz/docs/ExportingAPI.html

---

## 2. 方式一：Lua C Module（最成熟、推薦）

這是最務實的路徑：撰寫一個標準的 **Lua C Module**（C 共享函式庫），從 Cuberite 插件中以 Lua 的 `require()` 載入。C 模組可以執行任何原生程式碼，並透過 Lua C API 與 Cuberite 的插件環境互動。

### 2.1 實作步驟

#### 步驟 1：建立專案結構

```
MyPlugin/
├── main.lua          # Cuberite 插件主程式（Lua）
├── Info.lua          # 插件中繼資訊（選用）
├── libnative.so      # 編譯後的 C 模組
└── native/
    ├── native.c      # C 原始碼
    └── Makefile      # 編譯腳本
```

#### 步驟 2：撰寫 C 模組（native.c）

Cuberite 使用 **Lua 5.1 API**（不是 5.2+），因此必須使用 `luaL_register` 而非 `luaL_newlib`。

```c
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>

/* 可供 Lua 呼叫的 C 函式 */
static int native_hello(lua_State *L) {
    /* 會從 stack 讀取參數 */
    const char *name = luaL_checkstring(L, 1);
    lua_pushfstring(L, "Hello from C, %s!", name);
    return 1;  /* 回傳一個值 */
}

static int native_add(lua_State *L) {
    double a = luaL_checknumber(L, 1);
    double b = luaL_checknumber(L, 2);
    lua_pushnumber(L, a + b);
    return 1;
}

/* 函式註冊表 ── Lua 5.1 使用 luaL_register */
static const struct luaL_Reg mylib[] = {
    {"hello", native_hello},
    {"add",   native_add},
    {NULL, NULL}  /* 哨兵 */
};

/* 模組入口函式 ── 命名規則：luaopen_<模組名稱> */
int luaopen_native(lua_State *L) {
    /* Lua 5.1：同時註冊全域表並推至 stack */
    luaL_register(L, "native", mylib);
    return 1;
}
```

> **注意**：
> - 使用 `/usr/include/lua5.1/` 或 Cuberite 原始碼中的 `lib/lua/src/` 下的標頭
> - `luaopen_native` 必須以 `extern "C"`（C++）或直接（C）匯出

#### 步驟 3：從 Lua 插件中呼叫 C 模組（main.lua）

```lua
PLUGIN = nil

function Initialize(Plugin)
    Plugin:SetName("MyNativePlugin")
    Plugin:SetVersion(1)

    -- 載入 C 模組 ── 從同一目錄載入 native.so
    local ok, native = pcall(require, "native")
    if not ok then
        LOG("Failed to load native module: " .. tostring(native))
        return false
    end

    LOG(native.hello("Cuberite"))   -- 輸出 "Hello from C, Cuberite!"
    LOG("1 + 2 = " .. native.add(1, 2))  -- 輸出 "1 + 2 = 3"

    PLUGIN = Plugin
    return true
end
```

#### 步驟 4：編譯 C 模組

**Linux（.so）：**

```bash
# 使用系統的 Lua 5.1 開發套件
gcc -shared -fPIC -o libnative.so \
    -I/usr/include/lua5.1 \
    native.c \
    -llua5.1

# 或使用 Cuberite 原始碼中內建的 Lua 標頭
gcc -shared -fPIC -o libnative.so \
    -I/path/to/cuberite/lib/lua/src \
    native.c
```

**Windows（.dll）：**

```bash
gcc -shared -o native.dll \
    -I/path/to/cuberite/lib/lua/src \
    native.c \
    -llua51
```

> **關於連結 `-llua5.1`**：因為 Cuberite 以 `-rdynamic` 編譯，Linux 上通常**不需要**連結獨立的 `liblua.so`——符號會從 Cuberite 主執行檔解析。但若編譯器要求，可連結系統的 `liblua5.1`。

#### 步驟 5：部署

編譯產物（如 `libnative.so`）置於插件根目錄（與 `main.lua` 同層），Cuberite 啟動時即可載入。

### 2.2 進階：從 C 呼叫 Cuberite API

C 模組不僅可以供 Lua 呼叫，還可反過來操作 Cuberite 的 API。其做法是：從 Lua 端將 Cuberite 的 API 物件（如 `cWorld`、`cPlayer`）以 lightuserdata 或 userdata 的形式傳入 C 函式，C 函式再透過 Lua C API 操作這些物件。

#### 範例：從 C 設定方塊

```c
static int native_set_block(lua_State *L) {
    /* 預期參數：world物件, x, y, z, block_type, block_meta */
    /* 我們不直接操作 C++ 物件，而是回呼 Lua */
    lua_getglobal(L, "SetBlockFromC");
    if (lua_isfunction(L, -1)) {
        /* 將參數傳遞給 Lua 函式 */
        lua_pushvalue(L, 1);  /* world 物件 */
        lua_pushnumber(L, luaL_checknumber(L, 2));  /* x */
        lua_pushnumber(L, luaL_checknumber(L, 3));  /* y */
        lua_pushnumber(L, luaL_checknumber(L, 4));  /* z */
        lua_pushnumber(L, luaL_checknumber(L, 5));  /* block type */
        lua_call(L, 5, 0);
    }
    return 0;
}
```

```lua
-- 在 main.lua 中定義
function SetBlockFromC(World, X, Y, Z, BlockType)
    World:SetBlock(X, Y, Z, BlockType, 0)
end
```

### 2.3 實例參考：MCserver-RaspberryGPIO

現存最完整的實例是 **MCserver-RaspberryGPIO**[^gpio-plugin]，它展示了兩種方式：

- **純 Lua 版**：Lua 呼叫 `require("GPIO")` 載入 `rpi_gpio_lua` C 函式庫，透過 Cuberite 的 `cWorld:GetBlockInfo()` / `cWorld:SetBlock()` 操作方塊。
- **靜態 C 模組版**：額外載入自訂的 `MCmodule.so`（原始碼為 `main.c`[^gpio-mainc]），將方塊讀寫與 GPIO 邏輯全部搬到 C 中實作，大幅提昇效能。

編譯指令（來自該專案）：

```bash
gcc -Wall -shared -fPIC -o MCmodule.so \
    -I/usr/include/lua5.1 -llua5.1 \
    -std=c99 main.c -O2 -lwiringPi
```

[^gpio-plugin]: matemat13. (n.d.). MCserver-RaspberryGPIO. Retrieved 2026-09-13, from https://github.com/matemat13/MCserver-RaspberryGPIO
[^gpio-mainc]: matemat13. (n.d.). C module source/main.c. Retrieved 2026-09-13, from https://github.com/matemat13/MCserver-RaspberryGPIO/blob/master/Lua%20with%20static%20C%20module/C%20module%20source/main.c

### 2.4 策略：何時將程式碼搬進 C

效能敏感的場景可考慮將邏輯搬進 C 模組：

| 情境 | 建議 |
|---|---|
| 單純呼叫 Cuberite API 一次（如登入廣播） | 留在 Lua |
| 大量方塊操作（如世界編輯） | 搬進 C 模組 |
| 需要硬體存取（GPIO、序列埠） | 搬進 C 模組 |
| 既有 C/C++ 函式庫重複利用 | 包成 C 模組 |
| 高效能運算（碰撞檢測、路徑尋找） | 搬進 C 模組 |

---

## 3. 方式二：針對核心開發者的 tolua++ 繫結

此方式**並非給插件開發者**使用，而是 Cuberite 核心貢獻者將 C++ API 暴露給 Lua 的方式。但理解此機制有助於評估繫結策略。

### 3.1 自動繫結（tolua++）

在 C++ 標頭中以特殊註記標記要匯出的符號[^export-api]：

```cpp
// tolua_begin
class cPlayer
{
public:
    int GetHealth() const;  // 自動匯出
    // tolua_end
    void internalMethod();  // 不匯出
    // tolua_begin
    void SetHealth(int a_Health);  // 匯出
};
// tolua_end
```

tolua++ 解析這些標記，生成 `src/Bindings/Bindings.cpp`，在 `tolua_AllToLua_open()` 中一次註冊所有繫結。

### 3.2 手動繫結（ManualBindings*.cpp）

對回呼、UUID、陣列等複雜型別，核心開發者以 Lua C API 手動撰寫繫結函式[^manual-bindings]：

```cpp
static int tolua_cRoot_DoWithPlayerByUUID(lua_State * tolua_S)
{
    cLuaState L(tolua_S);
    if (
        !L.CheckParamSelf("cRoot") ||
        !L.CheckParamUUID(2) ||
        !L.CheckParamFunction(3) ||
        !L.CheckParamEnd(4)
    ) { return 0; }

    cRoot * Self;
    cUUID PlayerUUID;
    cLuaState::cRef FnRef;
    L.GetStackValues(1, Self, PlayerUUID, FnRef);

    bool res = Self->DoWithPlayerByUUID(PlayerUUID, [&](cPlayer & a_Player) {
        bool ret = false;
        L.Call(FnRef, &a_Player, cLuaState::Return, ret);
        return ret;
    });

    L.Push(res);
    return 1;
}
```

[^manual-bindings]: Cuberite Contributors. (n.d.). src/Bindings/ManualBindings.cpp. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/Bindings/ManualBindings.cpp

---

## 4. 方式三：未來展望——原生 C/C++ 插件系統

### 4.1 功能提案（Issue #5137）

2021 年，開發者 **GitAntoinee** 提出了一項功能請求[^issue-5137]：在 Cuberite 中新增原生 C/C++ 插件支援，以 `.dll` / `.so` 動態函式庫的形式載入插件。

**提議的架構**：

- 插件匯出 C 語言等級的函式（透過 `dlopen` / `dlsym` 或 `LoadLibrary` / `GetProcAddress` 載入）
  - `cuberite_initialize()` —— 插件初始化
  - `cuberite_on_disable()` —— 插件關閉
  - 以及對應各個 Hook 的函式
- C++ 使用者以 `extern "C"` 包裝
- 同時也為 Rust、D、Nim、Zig 等語言開啟可能性

**討論重點**：

| 支持方論點 | 反對方論點 |
|---|---|
| Cuberite 本身是 C++，原生插件才能發揮效能 | 可能變成單一使用者功能，後續無人維護成技術債 |
| 解鎖更多語言生態系（Rust、Zig 等） | ABI 穩定性問題——列舉型別無法像 Lua 那樣靈活調整 |
| C API 可作為其他語言繫結的基礎 | 跨平台分發複雜度高 |
| 插件活動低迷可能因不支援新版 Minecraft，非 Lua 問題 | 既有插件極少 |

**當前狀態**：仍為開啟的提案（`type/proposal`），未有人實作 PR。

有核心開發者（tigerw）建議折衷方案：實作一個 C Module 與既有 Lua API 繫結互動（即方式一），而非取代 Lua 插件系統。

[^issue-5137]: GitAntoinee. (2021-02-22). Write plugins to a library and not only lua. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/issues/5137

### 4.2 ABI 穩定性挑戰

若未來實現原生插件系統，須解決以下問題：

- **列舉值順序變更**：Lua 繫結無需擔心，但 C ABI 中列舉常數的重新排序會導致舊插件崩潰
- **版本依賴**：原生插件必須針對特定版本編譯，二進位相容性比 Lua 指令碼脆弱得多
- **跨平台分發**：需為 Linux（.so）、macOS（.dylib）、Windows（.dll）、ARM（Raspberry Pi）等分別編譯

---

## 5. 比較與建議

| 方案 | 複雜度 | 效能收益 | 可維護性 | 適用場景 |
|---|---|---|---|---|
| 純 Lua 插件 | 低 | 無 | 高 | 一般功能、命令繫結、事件處理 |
| Lua + C Module（方式一） | 中 | 高（關鍵路徑） | 中 | 高效能運算、硬體存取、既有函式庫重複利用 |
| 原生 C/C++ 插件（方式三） | 高（未實作） | 最高 | 低（需自行維護） | 目前不可行，關注 Issue #5137 |

### 建議策略

1. **優先以 Lua 實作**：Cuberite 的 Lua API 完整且文件化，多數插件需求已足夠。
2. **效能瓶頸以 C Module 加速**：對世界編輯、大量運算等場景，將熱點（hot path）搬進 C 模組。
3. **混合架構**：Lua 負責事件處理（hook）與命令繫結，C 模組負責運算密集或硬體相關工作。
4. **關注原生插件發展**：追蹤 GitHub Issue #5137，若未來有 PR 合併，屆時可評估遷移。

---

## 6. 參考資料

[^api-tutorial]: Cuberite Contributors. (n.d.). Writing a Cuberite plugin. Retrieved 2026-09-13, from https://api.cuberite.org/Writing-a-Cuberite-plugin.html

[^lua-version]: Cuberite Contributors. (n.d.). Cuberite API - Index. Retrieved 2026-09-13, from https://api.cuberite.org/

[^cpath-setup]: Cuberite Contributors. (n.d.). src/Bindings/PluginLua.cpp. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/Bindings/PluginLua.cpp

[^rdynamic]: Cuberite Contributors. (n.d.). src/CMakeLists.txt (lines 195-200). Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/CMakeLists.txt

[^export-api]: Cuberite Contributors. (n.d.). Exporting Cuberite API to Lua. Retrieved 2026-09-13, from http://cuberite.xoft.cz/docs/ExportingAPI.html

[^manual-bindings]: Cuberite Contributors. (n.d.). src/Bindings/ManualBindings.cpp. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/blob/master/src/Bindings/ManualBindings.cpp

[^gpio-plugin]: matemat13. (n.d.). MCserver-RaspberryGPIO. Retrieved 2026-09-13, from https://github.com/matemat13/MCserver-RaspberryGPIO

[^gpio-mainc]: matemat13. (n.d.). C module source/main.c. Retrieved 2026-09-13, from https://github.com/matemat13/MCserver-RaspberryGPIO/blob/master/Lua%20with%20static%20C%20module/C%20module%20source/main.c

[^issue-5137]: GitAntoinee. (2021-02-22). Write plugins to a library and not only lua (Issue #5137). Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/issues/5137