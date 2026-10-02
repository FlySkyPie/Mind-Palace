# 職業電子工作者的電電路資料管理方式：以連接器 Pin 指派為例

職業電子工程師與 PCB 設計者在處理電路資料時，核心挑戰在於「如何讓**邏輯連接**（哪個 pin 接哪個 pin）與**實體佈局**（實際 PCB 上的走線）維持同步」。以下整理業界標準做法。

## 1. Netlist（網絡清單）：所有 Pin 連接的單一事實來源

在專業 ECAD（Electronic CAD）軟體中，**Netlist** 是記錄所有網路（net）連接的主資料結構。每個 net 定義一條電氣連接，並精確對應到元件（component）的 reference designator 與 pin 編號。

典型 netlist 條目格式如 `PWR5V J1-1 U1-4`，表示名為 "PWR5V" 的 net 連接 J1 元件的 pin 1 到 U1 元件的 pin 4。[^altium-netlist]

Netlist 在 schematic capture（電路圖繪製）過程中**自動生成**——當工程師放置元件並繪製連接線時，CAD 軟體即時建立 netlist。

## 2. ECAD 軟體中的 Pin 管理功能

### Altium Designer（業界主流商用工具）

Altium Designer 具備完整的 **Pin/Part Swapping** 子系統：[^altium-pin-swapping]

- **Pin Swapping**：同一元件內屬於同一個 pin group 的 pin 可以互換。系統分析每個 pin 所屬的 net，在交換時動態重新指派 nets。
- **Differential Pair Swapping**：管理 FPGA 上差分對 pin 的交換。
- **Automatic Pin/Net Optimizer**：二階段最佳化器，透過重新指派 nets 來最小化交叉與連接長度。
- **FPGA Pin Mapper**：專門工具，將 FPGA 工具（如 Vivado）輸出的外部 pin 檔與 schematic 元件連接，比對兩域間的 pin 訊號。[^altium-fpga-pin]

### KiCad（開源替代工具）

KiCad 的 Eeschema 電路圖編輯器同樣透過 netlist 管理 pin 連接。Netlist 以 S-expression 格式儲存。當連線端點接近 pin 時，KiCad 自動建立 netlist 關聯。[^kicad-schematic]

## 3. 資料庫驅動的元件管理

大型工程組織不會手動重新輸入 pin 資料，而是使用 **Database Libraries**：

- **Altium DbLink 檔案（\*.DbLink）**：將 schematic 元件連結到公司中央資料庫（ODBC/OLE DB 相容，如 SQL Server、Access、Excel 等）。[^altium-dblink]
- **Altium DbLib 檔案（\*.DbLib）**：直接從資料庫放置元件，symbol、footprint 與參數資料即時建立。
- 連結後執行 **Tools → Update Parameters From Database** 即可將元件參數（含 pin 指派）與資料庫同步。

這是大型公司跨專案維持數百種 connector 類型一致性的核心方法。

## 4. 廠商中立交換格式

工程師使用標準格式交換 pin 指派資料：

| 格式 | 用途 |
|---|---|
| **IPC-D-356** | PCB 製造業標準 netlist 格式。包含 net 名稱、實體 pad 座標、層指派、元件 reference designator 與 pin 編號。CAM 工具用它比對 Gerber 資料確認連接正確性。[^ipc356] |
| **EDIF** | Electronic Data Interchange Format — 用於 schematic/pin 資料的廠商中立交換。 |
| **IPC-2581** | 業界聯盟標準，涵蓋 PCB 資料交換全流程。 |
| **WireList** | Netlist 匯出格式，以 reference designator 分組顯示 pin 編號間的連接關係。 |

## 5. 設計工作流程

職業工程師管理 pin 指派的典型流程：

1. **Schematic Capture** → 從元件庫放置元件。每個元件預先定義好 pin，附帶 designator、pin 名稱與選擇性的 pin group。
2. **Netlist 生成** → 繪製連線時，ECAD 工具建立 netlist 記錄每條 pin 連接。
3. **PCB Layout** → Netlist 驅動 "airwire"（rats nest）生成，顯示未繞線的連接。工程師據此在 PCB 上安排實體走線。
4. **Pin Swapping 最佳化** → 對複雜元件（如數百 pin 的 FPGA）執行自動最佳化器，重新指派 nets 以減少交叉連接。
5. **Design Update** → PCB 佈局變更後，透過 **Design Update** 程序同步回 schematic，保持 nets、pin 名稱與連接在邏輯域與實體域間一致。[^altium-schematic-tutorial]
6. **製造匯出** → 最終輸出 IPC-D-356 netlist（或等價格式）搭配 Gerber/ODB++ 製造資料供 CAM 驗證。

## 6. 輔助方法：交叉參照試算表

許多工程師在接手舊設計或進行板間連接驗證時，會維護**交叉參照試算表**。在 Altium Designer 中，Navigator Panel 允許工程師跨 sheet 追蹤 nets、驗證連接性，並匯出 netlist 資料進行外部審查。[^altium-navigator]

## 總結

| 方法 | 工具/格式 | 最佳用途 |
|---|---|---|
| Schematic netlist | Altium、KiCad、OrCAD | 自動追蹤每條 pin-to-pin 連接 |
| Pin/Part Swapping | Altium Designer | 最佳化 FPGA 與複雜連接器的 pin 指派 |
| 資料庫元件庫 | Altium DbLink/DbLib + ODBC | 跨專案維持 connector pinout 的單一事實來源 |
| IPC-D-356 netlist | 任何 ECAD → CAM | 製造環節驗證所有 pin 連接 |
| 交叉參照試算表 | 手動匯出 + CSV | 審查板間連接相容性 |
| FPGA Pin Mapper | Altium Designer | 同步 PCB net 名稱與 FPGA HDL 訊號名稱 |

## 參考文獻

[^altium-netlist]: Altium. (n.d.). What Are Netlists in PCB Design Projects. Retrieved 2026-10-01, from https://resources.altium.com/p/what-are-netlists-pcb-design-projects
[^altium-pin-swapping]: Altium. (n.d.). Swapping Pins, Pairs, and Parts. Retrieved 2026-10-01, from https://www.altium.com/documentation/altium-designer/sch-pcb/swapping-pins-pairs-parts
[^altium-fpga-pin]: Altium. (n.d.). Designing with the FPGA Pin Mapper. Retrieved 2026-10-01, from https://resources.altium.com/p/designing-fpga-pin-mapper
[^kicad-schematic]: KiCad. (n.d.). Schematic Capture. Retrieved 2026-10-01, from https://www.kicad.org/discover/schematic-capture/
[^altium-dblink]: Altium. (n.d.). Database Libraries — Linking Existing Components to a Database with a Database Link File. Retrieved 2026-10-01, from https://www.altium.com/documentation/altium-designer/components-libraries/database-libraries/linking-existing-components-database-link-file
[^ipc356]: IPC. (n.d.). IPC-D-356 — Bare Board Mechanical/Electrical Interface Standard for CCA. Retrieved 2026-10-01, from https://www.ipc.org/d-356
[^altium-schematic-tutorial]: Altium. (n.d.). Schematic Tutorial: A Journey of a Thousand PCBs. Retrieved 2026-10-01, from https://resources.altium.com/p/schematic-tutorial-altium-designer-journey-thousand-pcbs
[^altium-navigator]: Electronics StackExchange. (2023). Altium Designer — Is there a way to generate a list of all nets connected to an FPGA pin? Retrieved 2026-10-01, from https://electronics.stackexchange.com/questions/642871/altium-designer-is-there-a-way-to-generate-a-list-of-all-nets-connected-to-an