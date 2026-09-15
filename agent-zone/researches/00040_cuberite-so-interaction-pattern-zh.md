# 製作 .so 動態函式庫與 Cuberite 互動的方法

## 摘要

製作一個 `.so` 動態函式庫與 Cuberite 互動，核心機制是透過 Lua 的 C 模組協議：`.so` 必須實作一個 `luaopen_*` 函式作為進入點，由 Cuberite Plugin 的 Lua 程式碼以 `require("模組名")` 載入。載入後，`.so` 可以存取 Cuberite 的完整 Lua API、呼叫註冊的 C++ 綁定物件、註冊新的 C 函式供 Lua 使用，以及透過 Cuberite 內部的全域變數取得 Plugin 實例。

---

## 目錄

1. [基本架構：Lua C 模組協議](#1-基本架構lua-c-模組協議)
2. [編譯方式](#2-編譯方式)
3. [在 Plugin 中使用](#3-在-plugin-中使用)
4. [存取 Cuberite Lua API](#4-存取-cuberite-lua-api)
5. [註冊自訂 C 函式到 Lua](#5-註冊自訂-c-函式到-lua)
6. [取得 Cuberite Plugin 實例](#6-取得-cuberite-plugin-實例)
7. [完整範例](#7-完整範例)
8. [注意事項](#8-注意事項)

---

## 1. 基本架構：Lua C 模組協議

Lua 5.1 的 C 模組協議規定，一個原生模組必須：

1. **提供一個 `luaopen_*` C 函式**，其中 `*` 對應模組名稱。例如模組名為 `mymod`，則函式名為 `luaopen_mymod`。
2. **該函式簽名為** `int luaopen_mymod(lua_State *L)`
3. **函式應回傳 1**，並在 Lua stack 頂端留下一個 table（模組的命名空間）。
4. **必須使用 Lua 5.1 API** 編譯，因為 Cuberite 使用 Lua 5.1[^api-version]。

Cuberite 內部已包含兩個 C 模組作為範例：`sqlite3` (由 `lsqlite3` 提供) 和 `lxp` (Expat XML Parser)[^register-api]：

```cpp
void cLuaState::RegisterAPILibs(void)
{
    tolua_AllToLua_open(m_LuaState);       // tolua++ 自動綁定
    cManualBindings::Bind(m_LuaState);
    DeprecatedBindings::Bind(m_LuaState);
    cLuaJson::Bind(*this);
    luaopen_lsqlite3(m_LuaState);          // SQLite 原生 C 模組
    luaopen_lxp(m_LuaState);               // Expat XML Parser 原生 C 模組
}
```

## 2. 編譯方式

編譯 `.so` 時需要連結 Cuberite 使用的相同 Lua 5.1 標頭檔。Cuberite 的 Lua 原始碼位於 `lib/lua/src/`[^cuberite-repo]。編譯命令範例：

```bash
gcc -shared -fPIC -I/path/to/cuberite/lib/lua/src -o mymod.so mymod.c
```

關鍵要點：
- 使用 `-fPIC`（位置無關程式碼）
- 必須使用與 Cuberite 相同版本的 Lua 標頭（Lua 5.1）
- 無需連結 `liblua.so`，因為 `.so` 執行時會使用 Cuberite 進程內已載入的 Lua 符號
- 若你的 `.so` 依賴其他函式庫（如 `libpng`、`libcurl`），需一併連結

## 3. 在 Plugin 中使用

Cuberite 在 Plugin 載入時已將 Plugin 資料夾加入 `package.cpath`，支援 `?.so` 模式[^pluginlua-source]：

```cpp
m_LuaState.AddPackagePath("path", GetLocalFolder() + "/?.lua");
#ifdef _WIN32
    m_LuaState.AddPackagePath("cpath", GetLocalFolder() + "\\?.dll");
#else
    m_LuaState.AddPackagePath("cpath", GetLocalFolder() + "/?.so");
#endif
```

因此只要將 `.so` 放在 Plugin 資料夾中，Lua 程式碼即可直接使用 `require()` 載入：

```lua
-- 載入 mymod.so
local mymod = require("mymod")
```

Cuberite 會依序搜尋 `package.path`（`.lua`）與 `package.cpath`（`.so` / `.dll`），找到對應檔案後呼叫其 `luaopen_*` 函式。

## 4. 存取 Cuberite Lua API

`.so` 中的 C 程式碼可以完全存取 Cuberite 的 Lua 環境，因為它收到的 `lua_State *L` 就是 Plugin 的 Lua 狀態。這意味著：

### 4.1 標準 Lua 函式庫

所有標準 Lua 5.1 函式庫皆可透過 Lua C API 呼叫，包括 `package`、`io`、`os`、`string`、`math`、`table`、`debug` 等[^luastate-source]。

### 4.2 Cuberite API 物件

Cuberite 透過 `tolua++` 將大量 C++ 類別註冊為 Lua 中的 userdata。例如：

- `cPlayer` — 玩家物件
- `cWorld` — 世界物件  
- `cPluginManager` — Plugin 管理器
- `cRoot` — 根物件（存取伺服器核心）
- `cServer` — 伺服器設定
- `cBlockArea` — 區塊區域操作
- 以及其他約 116 個類別[^api]

在 `.so` 的 C 程式中，你可以透過 Lua 的 `lua_getglobal()` 取得這些物件，並使用 `tolua++` 的輔助函式操作它們。例如取得 `cRoot`：

```c
// 取得 cRoot 的 metatable 並調用其方法
lua_getglobal(L, "cRoot");
// 調用 cRoot:Get() 取得 singleton 實例
lua_getfield(L, -1, "Get");
lua_call(L, 0, 1);  // 現在 stack 頂端是 cRoot 實例
// 之後可以對此實例調用方法...
lua_pop(L, 1);  // 清理 stack
```

或者使用 `tolua++` 的 C++ 輔助函式 `tolua_tousertype()`：

```c
// 直接在 Lua 中執行 Cuberite API
lua_getglobal(L, "cRoot");
lua_getfield(L, -1, "Get");
lua_call(L, 0, 1);
void *root = tolua_tousertype(L, -1, "cRoot");
lua_pop(L, 1);
```

### 4.3 Hook 系統

Plugin 可以透過 `cPluginManager:AddHook()` 註冊事件處理。在 `.so` 中，你可以透過 Lua C API 呼叫這些函式，例如：

```c
// 取得 cPluginManager
lua_getglobal(L, "cPluginManager");
lua_getfield(L, -1, "Get");
lua_call(L, 0, 1);  // Stack: cPluginManager instance

// 呼叫 AddHook
lua_getfield(L, -1, "AddHook");
lua_pushvalue(L, -2);  // self
lua_pushnumber(L, cPluginManager_HOOK_TICK);  // HOOK_TICK 常數
lua_call(L, 2, 0);
lua_pop(L, 1);  // 清理 cPluginManager
```

或者你也可以在 `luaopen_*` 時直接在 C 中註冊 C 函式到 Lua 全域，然後在 Plugin 的 `.lua` 程式中使用這些函式。

## 5. 註冊自訂 C 函式到 Lua

`.so` 的 `luaopen_*` 函式中，可以註冊 C 函式供 Plugin 的 Lua 程式碼呼叫：

```c
static int my_c_function(lua_State *L) {
    double arg = luaL_checknumber(L, 1);
    lua_pushnumber(L, arg * 2.0);
    return 1;
}

int luaopen_mymod(lua_State *L) {
    lua_createtable(L, 0, 1);
    
    // 註冊 C 函式到 table 中
    lua_pushcfunction(L, my_c_function);
    lua_setfield(L, -2, "double_it");
    
    // 也可以註冊到全域
    lua_pushcfunction(L, my_c_function);
    lua_setglobal(L, "global_double_it");
    
    return 1;  // 回傳 table
}
```

在 Lua Plugin 中使用：

```lua
local mymod = require("mymod")
print(mymod.double_it(21))  -- 42
print(global_double_it(21)) -- 42
```

## 6. 取得 Cuberite Plugin 實例

Cuberite 在 Plugin 的 Lua 狀態中存放了兩個重要的全域變數：

1. **`_CuberiteInternal_PluginInstance`** — 型別為 `lightuserdata`，指向此 Plugin 的 `cPluginLua *` C++ 物件[^pluginlua-h]：

```cpp
lua_pushlightuserdata(m_LuaState, this);
lua_setglobal(m_LuaState, LUA_PLUGIN_INSTANCE_VAR_NAME);  // "_CuberiteInternal_PluginInstance"
```

在你的 `.so` 的 C 程式中：

```c
lua_getglobal(L, "_CuberiteInternal_PluginInstance");
cPluginLua *plugin = (cPluginLua *)lua_touserdata(L, -1);
lua_pop(L, 1);
```

2. **`g_Plugin`** — 型別為 `tolua++` 的 `cPluginLua` userdata，可從 Lua 層直接存取[^pluginlua-source]：

```cpp
tolua_pushusertype(m_LuaState, this, "cPluginLua");
lua_setglobal(m_LuaState, "g_Plugin");
```

## 7. 完整範例

以下是一個完整的 `.so` 範例，它註冊一個函式讓 Lua Plugin 可以呼叫，並展示如何與 Cuberite API 互動。

### C 程式碼 (`myhelper.c`)

```c
#include <stdio.h>
#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"

/* 在遊戲中廣播訊息 */
static int broadcast_message(lua_State *L) {
    const char *msg = luaL_checkstring(L, 1);
    
    // 透過 Cuberite API 取得 cRoot 實例
    lua_getglobal(L, "cRoot");
    lua_getfield(L, -1, "Get");
    lua_call(L, 0, 1);  // Stack: cRoot instance
    
    // 取得所有世界
    lua_getfield(L, -1, "ForEachWorld");
    lua_pushvalue(L, -2);  // self (cRoot)
    
    // 建立回呼函式
    lua_pushcfunction(L, function(L) {
        const char *world_name = lua_tostring(L, 1);
        printf("Broadcasting to world: %s\\n", world_name);
        return 0;
    });
    
    lua_call(L, 2, 1);  // ForEachWorld(self, callback) -> bool
    
    lua_pop(L, 1);  // 清理 cRoot
    
    lua_pushboolean(L, 1);
    return 1;
}

/* 計算兩點之間的距離（在 C 中執行，示範純運算） */
static int distance_3d(lua_State *L) {
    double x1 = luaL_checknumber(L, 1);
    double y1 = luaL_checknumber(L, 2);
    double z1 = luaL_checknumber(L, 3);
    double x2 = luaL_checknumber(L, 4);
    double y2 = luaL_checknumber(L, 5);
    double z2 = luaL_checknumber(L, 6);
    
    double dx = x1 - x2;
    double dy = y1 - y2;
    double dz = z1 - z2;
    
    lua_pushnumber(L, sqrt(dx*dx + dy*dy + dz*dz));
    return 1;
}

int luaopen_myhelper(lua_State *L) {
    static const luaL_Reg funcs[] = {
        {"broadcast_message", broadcast_message},
        {"distance_3d", distance_3d},
        {NULL, NULL}
    };
    
    luaL_register(L, "myhelper", funcs);
    return 1;
}
```

### Lua Plugin 使用範例

```lua
-- 載入 myhelper.so
local myhelper = require("myhelper")

-- 呼叫 C 函式
local dist = myhelper.distance_3d(0, 0, 0, 10, 10, 10)
LOG("Distance: " .. dist)

-- 呼叫操作 Cuberite API 的 C 函式
myhelper.broadcast_message("Hello from C!")
```

## 8. 注意事項

1. **Lua 5.1 API 相容性**：Cuberite 使用 Lua 5.1，你的 `.so` 必須使用 Lua 5.1 的 C API 編譯。Lua 5.2/5.3/5.4 的 API 變更（如 `luaL_register` 在 5.2+ 已被移除）可能導致不相容。

2. **執行緒安全**：Cuberite 是單執行緒的伺服器（每個 Lua 狀態有獨立的 mutex 保護）[^luastate-h]，`.so` 中的 C 函式不應建立自己的執行緒來操作 Lua 狀態。

3. **Plugin 資料夾**：`.so` 必須放在 Plugin 的資料夾中（或 `package.cpath` 涵蓋的路徑），才能被 `require()` 找到。

4. **不應連結外部 Lua**：`.so` 不應連結外部 `liblua.so`，應使用 Cuberite 進程內已載入的 Lua 符號。若連結外部 Lua 會導致兩個 Lua 實例共存，引發嚴重問題。

5. **記憶體管理**：透過 `tolua_newuserdata` 或 `lua_newuserdata` 分配的記憶體由 Lua GC 管理，不需手動釋放。

6. **與 tolua 物件的互動**：Cuberite 的 tolua 物件通常是 `full userdata`，需要使用 `tolua_tousertype()` 或 `tolua_pushusertype()` 系列函式來操作。

7. **除錯**：可以使用 `LOG()`、`LOGINFO()`、`LOGWARNING()` 系列函式（透過 `fmt` 函式庫），或者直接從 C 中調用 Lua 的 `print()`。

---

[^api-version]: Cuberite API Documentation — Globals class, `_VERSION = Lua 5.1`. Retrieved 2026-09-13, from https://api.cuberite.org/Globals.html

[^register-api]: cLuaState::RegisterAPILibs(). (n.d.). Cuberite source code, LuaState.cpp. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/LuaState.cpp

[^cuberite-repo]: Cuberite GitHub repository, file tree — `lib/lua/src/` contains Lua 5.1 source. Retrieved 2026-09-13, from https://github.com/cuberite/cuberite

[^pluginlua-source]: cPluginLua::Load(). (n.d.). Cuberite source code, PluginLua.cpp, showing `AddPackagePath` with `?.so` / `?.dll` and `g_Plugin` global. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/PluginLua.cpp

[^luastate-source]: cLuaState::Create() using `luaL_openlibs()`. (n.d.). Cuberite source code, LuaState.cpp. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/LuaState.cpp

[^api]: Cuberite API Documentation — Class index listing ~116 classes, plus tolua, sqlite3, lxp. Retrieved 2026-09-13, from https://api.cuberite.org/

[^pluginlua-h]: cPluginLua class header — `LUA_PLUGIN_INSTANCE_VAR_NAME` defined as `"_CuberiteInternal_PluginInstance"`. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/PluginLua.h

[^luastate-h]: cLuaState class header — `cLock` class providing `cCSLock` for thread safety. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/LuaState.h