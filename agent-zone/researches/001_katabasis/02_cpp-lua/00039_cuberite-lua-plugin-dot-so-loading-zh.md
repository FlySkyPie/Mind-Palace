# Cuberite Lua Plugin 能否載入動態函式庫（.so）

## 摘要

Cuberite 的 Lua Plugin 完全支援透過 Lua 標準 `require()` 函式載入動態函式庫（`.so` / `.dll`）。其 Lua 虛擬機不僅保留了完整的 `package` 標準函式庫（包含 `package.path`、`package.cpath`、`require`、`package.loaded`、`package.preload` 等），而且在載入 Plugin 時主動將 Plugin 資料夾加入 `package.cpath` 的搜尋路徑。此外，Cuberite 也透過 `tolua++` 將大量 C++ API 註冊為 Lua 全域物件，並內建了 SQLite (`sqlite3`) 與 Expat XML Parser (`lxp`) 兩個原生 C 模組作為範例。

---

## 關鍵發現

### 1. Lua 虛擬機初始化：所有標準函式庫均已開啟

Cuberite 在 `LuaState.cpp` 的 `Create()` 方法中調用 `luaL_openlibs()`，這會將所有 Lua 5.1 標準函式庫註冊到 Lua 狀態中，包括：

- `base`（`print`, `pairs`, `type`, `pcall`, `dofile` 等）
- `table`
- `io`（檔案 I/O）
- `os`（OS 功能，包含 `os.execute` 與 `os.clock` 等）
- `string`
- `math`
- `debug`
- **`package`**（模組載入機制，即 `require()`、`package.path`、`package.cpath`）

原始碼 (`LuaState.cpp`)[^luastate-source]：

```cpp
void cLuaState::Create(void)
{
    if (m_LuaState != nullptr) { return; }
    m_LuaState = lua_open();
    luaL_openlibs(m_LuaState);   // ← 所有標準函式庫全開
    m_IsOwned = true;
    // ...
}
```

### 2. Plugin 的 `package.cpath` 已包含 `.so` 路徑

在 `PluginLua.cpp` 的 `Load()` 方法中，Cuberite **明確**地將 Plugin 的本地資料夾加入 `package.cpath`，在非 Windows 系統上使用 `?.so` 模式，在 Windows 上使用 `?.dll` 模式[^pluginlua-source]：

```cpp
// Inject the identification global variables into the state:
lua_pushlightuserdata(m_LuaState, this);
lua_setglobal(m_LuaState, LUA_PLUGIN_INSTANCE_VAR_NAME);

// Add the plugin's folder to the package.path and package.cpath variables (#693):
m_LuaState.AddPackagePath("path", GetLocalFolder() + "/?.lua");
#ifdef _WIN32
    m_LuaState.AddPackagePath("cpath", GetLocalFolder() + "\\?.dll");
#else
    m_LuaState.AddPackagePath("cpath", GetLocalFolder() + "/?.so");
#endif
```

GitHub issue [#693][^issue693] 正是為了解決 Plugin 無法使用 `require()` 載入模組的問題，該 PR 在 `PluginLua.cpp` 中加入了上述 `AddPackagePath` 呼叫。

### 3. `AddPackagePath()` 實作：直接操作 `package` 表

`LuaState.cpp` 中的 `AddPackagePath()` 方法直接調用 Lua C API 來取得並修改 `package.path` 或 `package.cpath`[^luastate-source]：

```cpp
void cLuaState::AddPackagePath(const AString & a_PathVariable, const AString & a_Path)
{
    lua_getfield(m_LuaState, LUA_GLOBALSINDEX, "package");  // 取得 package 表
    lua_getfield(m_LuaState, -1, a_PathVariable.c_str());   // 取得 package.path 或 package.cpath
    // ... 附加新路徑 ...
    lua_setfield(m_LuaState, -2, a_PathVariable.c_str());   // 設定回 package 表
}
```

這證明了 `package` 表不僅存在且可正常存取。

### 4. `tolua++` C++ Binding 與內建原生模組

Cuberite 使用 `tolua++` 將大量 C++ 類別綁定至 Lua（如 `cPlayer`、`cWorld`、`cPluginManager` 等）[^api]：

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

這表明 Cuberite 本身就示範了如何透過 `luaopen_*` 函式註冊原生 C 模組，Plugin 開發者理應能夠用相同機制載入自己的 `.so`。

### 5. Lua 版本：Lua 5.1

Cuberite 使用 **Lua 5.1**（`_VERSION` 常數為 `"Lua 5.1"`）[^api]。Lua 5.1 的 `require()` 完全支援從 `package.cpath` 尋找 `.so` (Unix) 或 `.dll` (Windows) 動態函式庫。

---

## 結論

**Cuberite Lua Plugin 可以載入 `.so` 動態函式庫。** 因為：

1. Cuberite 使用 `luaL_openlibs()` 開啟了所有標準 Lua 函式庫，**包含 `package`**。
2. 每個 Plugin 載入時，其資料夾已被主動加入 `package.cpath`，並包含 `?.so` 模式。
3. Plugin 內的 `.lua` 檔案可直接使用 `require("你的函式庫")` 來載入放在 Plugin 資料夾中的 `.so` 檔案。
4. Cuberite 自身已經在 `RegisterAPILibs()` 中內建了 `sqlite3` 與 `lxp` 這兩個原生 C 模組，證明此機制運作正常。

## 注意事項

- `os` 與 `io` 函式庫也是可用的，Plugin 擁有較高的系統存取權限，設計時應考慮安全風險。
- 若要在 Plugin 中使用 `package.loadlib()` 或 `require()` 載入外部 `.so`，需要確認 `.so` 編譯時使用的 Lua API 版本與 Cuberite 的 Lua 5.1 相容。
- 提供了 `LUA_USE_POSIX` 定義（在 `PluginLua.cpp` 開頭），確保 Lua 的 POSIX 相關功能（如 `os.tmpname`）正常運作[^pluginlua-source]。
- 在 Windows 平台上對應為 `.dll`。

---

[^luastate-source]: cLuaState::Create() and AddPackagePath(). (n.d.). Cuberite source code, LuaState.cpp. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/LuaState.cpp

[^pluginlua-source]: cPluginLua::Load(). (n.d.). Cuberite source code, PluginLua.cpp. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/Bindings/PluginLua.cpp

[^issue693]: GitHub Issue #693 — Plugins should use require() for dependencies. (n.d.). Retrieved 2026-09-13, from https://github.com/cuberite/cuberite/issues/693

[^api]: Cuberite API Documentation — Globals class, `_VERSION = Lua 5.1`. (n.d.). Retrieved 2026-09-13, from https://api.cuberite.org/Globals.html