# C++ D-Bus 函式庫概覽

D-Bus 是一個用於 Linux 桌面環境與應用程式間的通訊協定（Inter-Process Communication，IPC），提供了應用程式之間傳送訊息與呼叫方法的標準機制。本文整理目前可用於 C++ 的 D-Bus 函式庫，涵蓋活躍維護中、穩定成熟與已不再維護的專案。

## 底層實作（C 語言函式庫，可從 C++ 直接使用）

### libdbus（參考實作）

libdbus 是 D-Bus 官方參考實作，以 C 語言撰寫。官方文件自身也警告：「如果你直接使用這個低階 API，你將面臨一些痛苦。」[^libdbus-pain]

- 授權條款：AFL/GPL
- 官方建議改用 GDBus、sd-bus 或 QtDBus 取代 libdbus[^dbusbindings]

### sd-bus（systemd）

sd-bus 是 systemd/libsystemd 的一部分，為 D-Bus 協定的獨立實作（而非包裝 libdbus）。

- API 文件：https://www.freedesktop.org/software/systemd/man/sd-bus.html
- 原始碼：https://github.com/systemd/systemd/

### GDBus（GLib）

GDBus 是 GNOME/GLib 專案的一部分（自 GLib 2.26+ 起），為 D-Bus 協定的獨立實作，不僅是 libdbus 的綁定。可透過 GObject Introspection 提供多語言綁定。[^dbusbindings]

- 程式語言：C（可從 C++ 使用）
- 授權條款：LGPL

## C++ 專用函式庫

### sdbus-c++（sdbus-cpp）

sdbus-c++ 是現代 C++17 的高階 D-Bus 函式庫，封裝 systemd 的 sd-bus，提供表達力強且易用的 API。是目前最活躍的現代 C++ D-Bus 選擇之一。[^sdbus-cpp]

- 標準需求：C++17
- 底層：sd-bus（libsystemd）
- 授權條款：LGPL v2.1+
- GitHub：https://github.com/Kistler-Group/sdbus-cpp
- 特色：現代 C++ 風格、表達力強的 API、基於 sd-bus

### dbus-cxx

dbus-cxx 是完全以 C++ 實作的 D-Bus 協定函式庫，不使用 libdbus，旨在提供緊湊的 C++ 風格介面並修正 libdbus 已知的多執行緒問題。使用 libsigc++ 提供物件導向的匯流排介面。[^dbus-cxx]

- 標準需求：C++17（最新版本）
- 底層：自實作協定（不依賴 libdbus）
- 授權條款：LGPL v3
- 網站：https://dbus-cxx.github.io/
- 特色：原生 C++ 實作、型別安全、執行緒安全、支援多種事件迴圈（Qt、GLib、libuv）、自動產生自省 XML、以 `sigc++` 處理回呼

### dbus-cpp

dbus-cpp 是 header-only 的 C++11 D-Bus 綁定，封裝 libdbus。由 Ubuntu/Canonical 的 Thomas Voß 維護。[^dbus-cpp-launchpad]

- 標準需求：C++11
- 底層：libdbus
- 授權條款：LGPL v3
- Launchpad：https://launchpad.net/dbus-cpp
- 狀態：僅有 header，輕量，曾用於 Ubuntu Touch 專案。最後上傳版本為 5.0.6（2026 年仍有更新）

### QtDBus

QtDBus 是 Qt 框架的一部分，為 libdbus 的高階 C++ 綁定。這是 Qt 專案中標準的 D-Bus 方案，成熟度與生態系最為完整。[^qtdbus]

- 標準需求：需搭配 Qt 使用
- 底層：libdbus
- 授權條款：LGPL v3 / GPL v2（Qt 商用授權）
- 文件：https://doc.qt.io/qt-6/qtdbus-module.html
- 特色：與 Qt 事件迴圈深度整合、QDBusInterface/QDBusConnection 等高階抽象、跨平台（但 D-Bus 模組僅限 Unix）

### dbus-c++（dbus-cplusplus，已停止維護）

dbus-c++ 是較早的 libdbus C++ 綁定，最後釋出於 2011 年，目前已不再維護，不建議新專案使用。[^dbusbindings]

- 狀態：已停止維護
- 網站：http://dbus-cplusplus.sourceforge.net/

## 比較摘要

| 函式庫 | C++ 標準 | 底層實作 | 授權條款 | 狀態 |
|--------|----------|---------|----------|------|
| sdbus-c++ | C++17 | sd-bus | LGPL v2.1+ | 活躍 |
| dbus-cxx | C++17 | 自實作 | LGPL v3 | 活躍 |
| dbus-cpp | C++11 | libdbus | LGPL v3 | 維護中 |
| QtDBus | 依 Qt | libdbus | LGPL/GPL | 穩定成熟 |
| dbus-c++ (cplusplus) | C++98 | libdbus | LGPL | 已停止 |

## 選擇建議

- **新專案、無 Qt 相依**：優先考慮 **sdbus-c++**（現代 C++17、基於 sd-bus）或 **dbus-cxx**（自實作協定、無外部依賴、型別安全）
- **使用 Qt 的專案**：直接使用 **QtDBus**，與 Qt 生態系整合最佳
- **需要極輕量方案**：**dbus-cpp**（header-only、C++11 友善）
- **不建議**：dbus-c++（dbus-cplusplus），因已多年未維護

## Reference

[^libdbus-pain]: freedesktop.org. (n.d.). D-Bus API documentation. Retrieved 2026-09-13, from https://dbus.freedesktop.org/doc/api/html/index.html

[^dbusbindings]: freedesktop.org. (n.d.). DBusBindings. Retrieved 2026-09-13, from https://www.freedesktop.org/wiki/Software/DBusBindings/

[^sdbus-cpp]: Kistler Group. (n.d.). sdbus-c++: High-level C++ D-Bus library for Linux. Retrieved 2026-09-13, from https://github.com/Kistler-Group/sdbus-cpp

[^dbus-cxx]: dbus-cxx authors. (n.d.). dbus-cxx Library. Retrieved 2026-09-13, from https://dbus-cxx.github.io/

[^dbus-cpp-launchpad]: Thomas Voß. (2013). dbus-cpp: A header-only dbus-binding leveraging C++-11. Retrieved 2026-09-13, from https://launchpad.net/dbus-cpp

[^qtdbus]: Qt Project. (n.d.). Qt D-Bus C++ Classes - Qt 6 Documentation. Retrieved 2026-09-13, from https://doc.qt.io/qt-6/qtdbus-module.html