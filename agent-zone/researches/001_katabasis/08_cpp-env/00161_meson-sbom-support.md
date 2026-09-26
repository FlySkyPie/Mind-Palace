# Meson 建置系統能否產生軟體物料清單 (SBOM)？

## 概述

**簡短答案：Meson 本身沒有內建 SBOM 產生功能，但可以透過第三方工具達成。**

Meson（版本 1.x）並未提供原生 SBOM 產生能力。然而，Meson 在建置過程中會產生豐富的內省資料（introspection data），第三方工具可以利用這些資料來產生標準格式（如 CycloneDX）的 SBOM。相較之下，CMake（實驗性支援）和 Bazel 在這方面已有更成熟的整合。

---

## 1. Meson 有內建 SBOM 功能嗎？

**沒有。** Meson 官方文件、GitHub 儲存庫的 issues 或 PR 中皆未提及 SBOM 支援。Meson 的官方比較頁面（與 CMake、Bazel 等比較）也未將 SBOM 列為區別項目。[^comparisons]

**不過，** Meson 在執行 `meson setup` 後，建置目錄下的 `meson-info/` 資料夾會產生數個 JSON 內省檔案，可作為 SBOM 的基礎：[^ide-int]

- **`intro-dependencies.json`** — 列出所有已發現的相依套件，含名稱、版本及編譯/連結參數
- **`intro-projectinfo.json`** — 專案後設資料（名稱、版本等）
- **`intro-targets.json`** — 所有建置目標及其外部相依套件名稱

此外，`meson introspect --scan-dependencies /path/to/meson.build` 可在不需建置目錄的情況下掃描相依套件。

---

## 2. 第三方工具

目前有兩個開源工具可為 Meson 專案產生 SBOM：

### a) mesonsbom

- **儲存庫**：<https://github.com/Framstag/mesonsbom>
- **授權**：GPL-3.0-or-later
- **語言**：C++
- **功能**：讀取 `meson-info/intro-projectinfo.json` 及 `intro-dependencies.json`，產生 **CycloneDX 1.6 JSON** 格式的 SBOM
- **特色**：支援全專案 SBOM 及**特定目標 SBOM**（`--target` 參數）
- **狀態**：作者已明確表示此為個人專案，與官方 Meson 專案無關。[^mesonsbom-issue]

### b) embtrace-sbom

- **儲存庫**：<https://github.com/Innomatica-GmbH/embtrace-sbom>
- **授權**：GPL-3.0-or-later
- **功能**：專為**嵌入式系統建置**設計，支援 CMake、Make、**Meson**、Autotools、Yocto、Buildroot、Zephyr 及 vcpkg
- **輸出**：CycloneDX 1.6 SBOM
- **優勢**：能處理 C/C++ 嵌入式專案的相依套件——這類專案在主流 SBOM 掃描器（如 Syft）中通常回傳零結果。[^embtrace-sbom]

### c) 商業方案：Safeguard

- **網站**：<https://safeguard.sh/>
- **功能**：擷取 Meson 內省資料（`meson introspect --all`）產生 CycloneDX 1.5 SBOM，並可強制要求 wrap 檔案的雜湊值。[^safeguard]

### d) 通用 SBOM 掃描器（Syft、cdxgen、Trivy）

**不支援 Meson。** 這些工具主要針對 npm、PyPI、Go、Cargo 等套件生態系，無法解析 Meson 建置定義檔。embtrace-sbom 的 README 報告指出 Syft 1.51.1 對嵌入式 Meson/CMake 專案回傳零結果。[^embtrace-sbom]

---

## 3. 與其他建置系統比較

| 建置系統 | 內建 SBOM 支援 | 社群工具 | 可用格式 |
|---|---|---|---|
| **Meson** | ❌ 無 | mesonsbom、embtrace-sbom | CycloneDX 1.6 |
| **CMake** | ✅ **實驗性**（2026年起）[^kitware] | cmake-sbom（DEMCON） | SPDX 3.0.1（原生）、SPDX 2.3（cmake-sbom） |
| **Bazel** | ✅ rules_license、sbom-tool[^sbom-tool] | 多種成熟選項 | SPDX 2.3、CycloneDX 1.6 |
| **Autotools** | ❌ 無 | 極少 | N/A |

### 重點差異

- **CMake** 是這方面的領導者。Kitware 於 2026 年 9 月宣布實驗性原生 SBOM 產生功能，專案只需設定 `CMAKE_EXPERIMENTAL_GENERATE_SBOM` 並使用 `install(SBOM ...)` 或 `export(SBOM ...)`，即可從 CMake 的內部相依圖直接產生 SPDX 3.0.1 JSON-LD SBOM。[^kitware] DEMCON 的 `cmake-sbom` 模組則提供更成熟的 SPDX 2.3 產生能力。[^cmake-sbom]

- **Bazel** 有多種選擇：`rules_license`（SPDX）、Eclipse 的 `sbom-tool`（SPDX 2.3 + CycloneDX 1.6）以及 `rules_sbom`（Katenaria）。

- **Meson** 完全依賴第三方工具讀取其內省資料，整合程度不如 CMake 或 Bazel。

---

## 4. 相關資源

| 資源 | 網址 | 相關性 |
|---|---|---|
| mesonsbom 命名詢問（Issue #15775） | <https://github.com/mesonbuild/meson/issues/15775> | 社群工具作者詢問 Meson 團隊，確認官方無 SBOM 工具 |
| Meson IDE 整合文件 | <https://mesonbuild.com/IDE-integration.html> | 說明 `meson-info/*.json` 內省檔案格式 |
| GN vs Meson 安全性比較（Safeguard） | <https://safeguard.sh/resources/blog/gn-meson-build-system-security-comparison> | 分析 Meson 供應鏈安全，涵蓋 wrap 檔案及內省資料 |
| Meson Wrap 相依系統 | <https://mesonbuild.com/Wrap-dependency-system-manual.html> | Wrap 系統支援雜湊驗證下載，與 SBOM 完整性相關 |
| Kitware: CMake SBOM 產生 | <https://www.kitware.com/generating-sboms-with-cmake/> | CMake 原生 SBOM 支援的官方說明（供比較用） |
| Meson 討論串 #13822 | <https://github.com/mesonbuild/meson/discussions/13822> | 討論利用 `meson-info/*.json` 在不建置的情況進行相依分析 |

---

## 5. 結論

Meson 無法原生產生 SBOM，但其 `meson-info/` JSON 內省資料層提供了紮實的基礎，使第三方工具可以輕鬆產生 SBOM。社群工具 **mesonsbom** 補上了 CycloneDX 格式的缺口；**embtrace-sbom** 則為嵌入式專案提供更廣泛的覆蓋。Meson 在這方面落後於 CMake（實驗性原生支援）和 Bazel（多種成熟方案），但可用的社群工具足以滿足基本需求。

[^comparisons]: Meson build system. (n.d.). *Comparison with other build systems*. Retrieved 2026-09-25, from https://mesonbuild.com/Comparisons.html
[^ide-int]: Meson build system. (n.d.). *IDE integration*. Retrieved 2026-09-25, from https://mesonbuild.com/IDE-integration.html
[^mesonsbom-issue]: mesonbuild/meson. (n.d.). *Issue #15775: Question: Is it okay to name a tool "mesonsbom"?*. Retrieved 2026-09-25, from https://github.com/mesonbuild/meson/issues/15775
[^embtrace-sbom]: Innomatica GmbH. (n.d.). *embtrace-sbom: Generate CycloneDX SBOMs for embedded builds*. Retrieved 2026-09-25, from https://github.com/Innomatica-GmbH/embtrace-sbom
[^safeguard]: Safeguard. (n.d.). *Secure GN and Meson builds with automated SBOMs, policy enforcement and vulnerability detection*. Retrieved 2026-09-25, from https://safeguard.sh/resources/blog/gn-meson-build-system-security-comparison
[^kitware]: Kitware. (2026-09-08). *Generating SBOMs with CMake*. Retrieved 2026-09-25, from https://www.kitware.com/generating-sboms-with-cmake/
[^cmake-sbom]: DEMCON. (n.d.). *cmake-sbom: Generate SPDX SBOM documents for your CMake project*. Retrieved 2026-09-25, from https://github.com/DEMCON/cmake-sbom
[^sbom-tool]: Eclipse Foundation. (n.d.). *sbom-tool: Generate SBOMs for Eclipse projects*. Retrieved 2026-09-25, from https://github.com/eclipse-score/sbom-tool