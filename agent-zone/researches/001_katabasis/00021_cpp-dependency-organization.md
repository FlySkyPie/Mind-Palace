# C++ 依賴管理與組織方式研究

## 摘要

本報告探討現代 C++ 專案中依賴管理（Dependency Management）的主流方案，涵蓋專案結構設計、套件管理器（Conan、vcpkg）、CMake 內建機制（FetchContent、find_package），以及 git submodule 等傳統做法。針對不同規模與場景，提供決策建議與最佳實踐。

## 1. 問題背景

C++ 缺乏官方統一的套件管理系統（不像 Rust 的 Cargo、npm、Python 的 pip），導致依賴管理方式多元且碎片化。選擇不當會導致：

- 建置可重現性（Reproducibility）問題
- Diamond dependency 衝突（A→C, B→C 不同版本）
- 跨平台移植困難
- 供應鏈安全風險

## 2. 主流依賴管理方案

### 2.1 CMake FetchContent（內建、從原始碼建置）

CMake 3.11+ 提供的 FetchContent 模組，可在 configure 階段從遠端下載依賴並整合進建置流程中[^cmake-fetchcontent]。

```cmake
include(FetchContent)
FetchContent_Declare(
  spdlog
  GIT_REPOSITORY https://github.com/gabime/spdlog.git
  GIT_TAG        v1.13.0
  GIT_SHALLOW    TRUE
  SYSTEM
)
FetchContent_MakeAvailable(spdlog)
```

**優勢**：
- 零外部工具依賴（只需 CMake）
- 版本釘選精確（tag / commit SHA）
- 與建置流程深度整合

**劣勢**：
- 每次都是從原始碼編譯，初次建置慢
- 無中央快取機制（除非配合 CPM.cache）
- 複雜的傳遞依賴圖易有衝突

### 2.2 vcpkg（Microsoft 維護的套件管理器）

vcpkg 是跨平台的 C/C++ 套件管理器，提供超過 2000 個 port（套件建置腳本）[^vcpkg-overview]。**Manifest mode**（透過 `vcpkg.json`）是官方推薦的做法[^vcpkg-manifest]。

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "dependencies": [
    "fmt",
    {
      "name": "cpprestsdk",
      "default-features": false
    }
  ]
}
```

整合 CMake 時透過 Toolchain file：

```json
{
  "version": 3,
  "configurePresets": [{
    "name": "default",
    "cacheVariables": {
      "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
    }
  }]
}
```

**優勢**：
- 大量現成 port（超過 2000 個）
- Manifest mode 支援版本控管、隔離的依賴目錄
- 支援客製 registry 與 overlay ports
- Microsoft 持續維護

**劣勢**：
- 初次下載 port 腳本較慢
- 版本選擇採 Minimum Version Selection，行為與傳統 semver 不同
- CI 環境需額外設定 vcpkg 快取

### 2.3 Conan（去中心化套件管理器）

Conan 是去中心化的 C/C++ 套件管理器，使用 Python 寫的 `conanfile.py` 描述套件[^conan-intro]。

```python
from conan import ConanFile
class HelloRecipe(ConanFile):
    name = "hello"
    version = "1.0"
    settings = "os", "compiler", "build_type", "arch"
    options = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}
    package_type = "library"

    def requirements(self):
        self.requires("fmt/[>=10.0 <11]")

    def layout(self):
        cmake_layout(self)

    def generate(self):
        deps = CMakeDeps(self)
        deps.generate()
        tc = CMakeToolchain(self)
        tc.generate()
```

**優勢**：
- 支援二進位預編譯，重複建置快
- 精細的套件選項控制（shared/static/fPIC）
- 支援版本範圍（Version Ranges）語法
- 成熟的私有儲存庫方案（Artifactory、conan_server）
- 平台與編譯器感知

**劣勢**：
- 學習曲線較陡（Python recipe）
- 需要額外工具安裝（conan CLI）
- 團隊需制定 profile 管理策略

### 2.4 CPM.cmake（FetchContent 的封裝層）

CPM.cmake 是 FetchContent 的薄封裝，提供更簡潔的語法與快取機制[^cpm-cmake]。

```cmake
include(cmake/CPM.cmake)
CPMAddPackage("gh:fmtlib/fmt#7.1.3")
CPMAddPackage("gh:nlohmann/json@3.10.5")
```

### 2.5 Git Submodule（傳統方式）

最原始的依賴管理方式，直接將第三方庫作為 git submodule 納入版本控制。

**優勢**：版本精確、無外部工具、離線可用。
**劣勢**：更新麻煩、無傳遞依賴管理、倉庫膨脹。

## 3. 專案結構建議

### 3.1 中型專案推薦結構[^cmake-proj-structure]

```
project-root/
├── CMakeLists.txt           # 根目錄建置檔
├── CMakePresets.json        # CMake preset 設定
├── vcpkg.json               # vcpkg manifest（若使用 vcpkg）
├── conanfile.py             # Conan recipe（若使用 Conan）
├── cmake/                   # 共用 CMake modules
│   ├── CompilerWarnings.cmake
│   └── Config.cmake.in
├── include/<project>/       # 公開標頭檔
│   └── module/
│       └── api.hpp
├── src/                     # 實作原始碼
│   └── module/
│       └── impl.cpp
├── tests/                   # 單元測試
│   ├── CMakeLists.txt
│   └── test_module.cpp
├── examples/                # 使用範例
│   └── demo.cpp
└── third_party/             # 第三方原始碼（submodule 或 FetchContent 下載）
    └── ...
```

### 3.2 依賴組織原則

根據 CMake 官方指南與社群慣例，依賴宣告應遵守以下原則[^cmake-dependency-guide]：

1. **由上而下宣告**：最上層專案應宣告所有（含傳遞性）依賴的版本，子專案不得覆寫
2. **Diamond dependency 處理**：利用 CMake 的「first to declare, wins」規則，父專案先宣告所有子依賴
3. **Build vs Host 區分**：工具鏈依賴（cmake、ninja）與函式庫依賴應明確分離
4. **條件式依賴**：平台感知（`WIN32`、`APPLE`）、功能開關

```cmake
# FetchContent 的 diamond dependency 正確做法
# 先宣告所有依賴，再統一 MakeAvailable
FetchContent_Declare(fmt ...)
FetchContent_Declare(spdlog ...)
FetchContent_Declare(myapp_dep ...)

# 依賴順序：fmt 必須在 spdlog 之前
FetchContent_MakeAvailable(fmt spdlog myapp_dep)
```

## 4. 方案比較與決策矩陣

| 面向 | FetchContent | vcpkg | Conan | Git Submodule |
|------|-------------|-------|-------|---------------|
| 外部工具依賴 | 無（僅 CMake） | vcpkg CLI | Conan CLI | Git |
| 初次建置速度 | 慢（原始碼編譯） | 中（下載 port） | 快（二進位） | 中 |
| 可重現性 | 高（SHA pin） | 高（baseline） | 高（lockfile） | 高（commit pin） |
| 傳遞依賴管理 | 手動管理 | 自動 | 自動 | 無 |
| 私有支援 | Git repo | Custom registry | Artifactory | 直接 |
| 學習曲線 | 低 | 中 | 高 | 低 |
| 供應鏈安全 | URL hash | Baseline checksum | Checksum | Git commit |
| 跨平台 | CMake 支援即可 | Win/Lin/Mac | 全平台 | 全平台 |
| 社群生態 | 分散 | 2000+ ports | ConanCenter | 無 |

## 5. 場景建議

### 小型專案（< 10 個依賴）

- **推薦方案**：FetchContent + CPM.cmake
- 理由：零額外工具，CMake 內建，學習成本最低

### 中型團隊（10-50 個依賴，多平台）

- **推薦方案**：vcpkg manifest mode
- 理由：Manifest 清晰，Toolchain 整合簡單，Microsoft 支援穩定

### 大型專案（50+ 依賴，跨團隊，私有套件）

- **推薦方案**：Conan
- 理由：精細的選項控制，二進位快取，私有儲存庫成熟

### 重視可重現性與安全的 CI 環境

- **混合策略**：結合 FetchContent（`FIND_PACKAGE_ARGS`，CMake 3.24+）+ vcpkg 或 Conan 作為 Dependency Provider

```cmake
# CMake 3.24+ 提供的 Hybrid 模式
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        703bd9caab50b139428cea1aaff9974ebee5742e
  FIND_PACKAGE_ARGS NAMES GTest
)
FetchContent_MakeAvailable(googletest)
# 先找系統安裝的 GTest，找不到才從原始碼建置
```

## 6. 結論

C++ 的依賴管理已從早期各自為政的階段走向成熟：

1. **小型專案**優先選 FetchContent/CPM.cmake
2. **中大型專案**選 vcpkg（簡單易用）或 Conan（功能完整）
3. **無論選哪種方案**，都應遵守「根專案控制版本」「釘選具體版本而非浮動標籤」「區分宿主與工具鏈依賴」的原則
4. 版本鎖定（Lockfile / Baseline / Override）是確保可重現性的關鍵機制
5. 供應鏈安全方面，應驗證 checksum、釘選 commit SHA、避免執行未審查的建置腳本

## 參考資料

[^cmake-fetchcontent]: CMake. (n.d.). FetchContent. Retrieved 2026-09-13, from https://cmake.org/cmake/help/latest/module/FetchContent.html

[^vcpkg-overview]: Microsoft. (n.d.). vcpkg: C++ Library Manager. Retrieved 2026-09-13, from https://learn.microsoft.com/en-us/vcpkg/

[^vcpkg-manifest]: Microsoft. (n.d.). Manifest mode. Retrieved 2026-09-13, from https://learn.microsoft.com/en-us/vcpkg/concepts/manifest-mode

[^conan-intro]: Conan. (n.d.). Conan 2 Introduction. Retrieved 2026-09-13, from https://docs.conan.io/2/introduction.html

[^cpm-cmake]: CPM.cmake. (n.d.). CMake's Missing Package Manager. Retrieved 2026-09-13, from https://github.com/cpm-cmake/CPM.cmake

[^cmake-dependency-guide]: CMake. (n.d.). Using Dependencies Guide. Retrieved 2026-09-13, from https://cmake.org/cmake/help/latest/guide/using-dependencies/index.html

[^cmake-proj-structure]: Zafar, W. (n.d.). CMake Mastery Part 10: Managing Dependencies with FetchContent. Retrieved 2026-09-13, from https://www.wasilzafar.com/pages/series/cmake-mastery/cmake-mastery-part10-fetchcontent.html

[^awesome-cpp]: fffaraz. (n.d.). awesome-cpp: A curated list of awesome C++ frameworks, libraries, resources. Retrieved 2026-09-13, from https://github.com/fffaraz/awesome-cpp

[^conan-guidelines]: Conan. (n.d.). Conan Guidelines. Retrieved 2026-09-13, from https://docs.conan.io/2/knowledge/guidelines.html