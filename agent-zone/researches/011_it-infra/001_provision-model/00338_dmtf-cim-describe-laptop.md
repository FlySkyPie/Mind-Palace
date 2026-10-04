# DMTF CIM（Common Information Model）能否描述「一台筆記型電腦」？

## 概要

DMTF（Distributed Management Task Force）所制定的 CIM（Common Information Model）是一套開放、可擴充、物件導向的資訊管理標準，以 UML 為基礎定義 IT 資源的共通表示方式。本報告確認：**CIM 完全能夠描述一台筆記型電腦的整體與各子系統**，並有對應的專用規範（DASH）及數十個相關 Profile 為之支援。

## DMTF CIM 簡介

CIM 包含三個部分[^cim-what]：

1. **CIM Schema** — 類別、屬性、關聯與繼承層次的實際定義
2. **CIM Infrastructure Specification** — 架構、語言、與其他管理模型（如 SNMP）的對應方式
3. **CIM Metamodel** — 定義建構新合規模型的語意

CIM 是 DMTF 多數管理標準的基礎模型，涵蓋「電腦系統、作業系統、網路、中介軟體、服務與儲存」等管理範疇[^cim-wiki]。其具體實作包括 Microsoft WMI（Windows Management Instrumentation）、Intel AMT（Active Management Technology）、SBLIM（Standards Based Linux Instrumentation for Manageability）及 OpenLMI 等[^cim-impl]。

## 描述筆記型電腦的核心類別

| 筆記型電腦元件 | CIM 類別 | 關鍵屬性 |
|---|---|---|
| **筆記型電腦本體** | `CIM_UnitaryComputerSystem`（繼承 `CIM_ComputerSystem`） | `Dedicated = 33`（"Laptop"）[^dedicated]；另有 `Name`、`ResetCapability`、`PowerManagementCapabilities` 等 |
| **物理外殼/機殼** | `CIM_Chassis` | `ChassisPackageType = 9`（"LapTop"）或 `10`（"Notebook"）[^chassis]；亦有 `Height`、`Width`、`Depth`、`Weight`、`Manufacturer`、`SerialNumber`、`Model` |
| **CPU** | `CIM_Processor` | `CurrentClockSpeed`、`MaxClockSpeed`、`Family`、`Stepping`、`NumberOfEnabledCores`、`AddressWidth`、`Characteristics`（64-bit、Virtualization 等）[^processor] |
| **記憶體（邏輯）** | `CIM_Memory`（子類 `CIM_VolatileMemory`） | `StartingAddress`、`EndingAddress`、`Volatile`[^memory] |
| **記憶體（物理 DIMM）** | `CIM_PhysicalMemory`（繼承 `CIM_Chip`） | `Capacity`（bytes）、`MemoryType`（DDR–DDR5、LPDDR5 等）、`Speed`、`FormFactor`（SIMM、DIMM、SODIMM）、`BankLabel`、`ConfiguredMemoryClockSpeed`[^physical-memory] |
| **儲存裝置** | `CIM_DiskDrive`（繼承 `CIM_MediaAccessDevice`） | 代表 OS 可見的物理磁碟（SSD/HDD）[^diskdrive] |
| **顯示器（內建）** | `CIM_FlatPanel`（抽象類 `CIM_Display` 的子類） | `BuiltIn = True`（直接附屬於可攜式電腦）[^flatpanel]；`DisplayTechnologyType`（LCD、OLED 等）、`CurrentResolutionH`、`CurrentResolutionV`、`Brightness`、`Contrast`、`LightSource`（Backlit、Edgelit、Reflective）[^display] |
| **鍵盤** | `CIM_Keyboard` | 存在於 CIM Schema 中[^keyboard] |
| **觸控板** | `CIM_PointingDevice` | `Handedness`、`NumberOfButtons`[^pointing] |
| **電池** | `CIM_Battery`（繼承 `CIM_PowerSource`） | `Chemistry`（Li-ion、Li-Polymer 等）、`DesignCapacity`、`DesignVoltage`、`EstimatedChargeRemaining`、`EstimatedRunTime`、`FullChargeCapacity`、`HealthPercent`、`RechargeCount`、`BatteryStatus`（Fully Charged、Charging、Low、Critical）、`ChargingStatus`（Charging／Discharging／Idle）[^battery] |
| **網路（有線）** | `CIM_EthernetPort` | MAC 位址、連結狀態、IP 設定 |
| **網路（無線）** | `CIM_WiFiPort` | 無線介面管理[^wifi] |
| **BIOS/UEFI** | `CIM_BIOSElement`、`CIM_BIOSFeature` | 韌體版本、開機設定 |
| **作業系統** | `CIM_OperatingSystem` | OS 類型、版本、安裝日期 |
| **電源供應器（AC 變壓器）** | `CIM_PowerSupply` | 額定輸出功率、輸入電壓[^psu] |
| **散熱風扇** | `CIM_Fan` | 轉速、狀態[^fan] |
| **感應器** | `CIM_Sensor`、`CIM_TemperatureSensor`、`CIM_VoltageSensor` | 溫度、電壓讀值[^sensor] |
| **主機板** | `CIM_Card` | 實體插槽佈局、尺寸 |

## DASH 規範：筆記型電腦管理的事實標準

DMTF 的 **DASH（Desktop and mobile Architecture for System Hardware）** 是一套專為桌上型與行動用戶端系統（即桌機與筆電）設計的管理規範。DASH 官方說明指出：

> "DMTF's Desktop and mobile Architecture for System Hardware (DASH) Standard is a suite of specifications that takes full advantage of DMTF's Web Services for Management (WS-Management) specification – delivering standards-based web services management for **desktop and mobile client systems**." [^dash]

DASH 支援 KVM（Keyboard、Video、Mouse）重新導向、文字主控台重新導向、USB 及媒體重新導向，以及 BIOS、電池、NIC、MAC、IP 位址等管理功能[^dash]。

## 相關 DMTF Profile

以下 Profile 定義了管理筆記型電腦各子系統時應使用哪些 CIM 類別及其行為方式[^profiles]：

| Profile 編號 | 名稱 | 管理範疇 |
|---|---|---|
| **DSP1058** | Base Desktop and Mobile Profile | 桌機與筆電的核心 Profile |
| **DSP1052** | Computer System Profile | 電腦系統整體管理 |
| **DSP1108** | Physical Computer System View Profile | 電腦系統實體視圖 |
| **DSP1022** | CPU Profile | 處理器監控 |
| **DSP1026** | System Memory Profile | 記憶體監控 |
| **DSP1030** | Battery Profile | 電池管理 |
| **DSP1061** | BIOS Management Profile | BIOS/UEFI 設定管理 |
| **DSP1011** | Physical Asset Profile | 機殼、主機板等實體資產管理 |
| **DSP1027** | Power State Management Profile | 電源狀態轉換（休眠、暫停等） |
| **DSP1085** | Power Utilization Management Profile | 功耗監控 |
| **DSP1015** | Power Supply Profile | 電源供應器／AC 變壓器 |
| **DSP1013** | Fan Profile | 散熱管理 |
| **DSP1009** | Sensors Profile | 感應器監控 |
| **DSP1014** | Ethernet Port Profile | 有線網路管理 |
| **DSP1088** | Wi-Fi Port Profile | 無線網路管理 |
| **DSP1075** | PCI Device Profile | 內部裝置列舉 |
| **DSP1025** | Software Update Profile | 韌體／軟體更新 |
| **DSP1023** | Software Inventory Profile | 安裝軟體清查 |
| **DSP1029** | OS Status Profile | 作業系統狀態 |
| **DSP1012** | Boot Control Profile | 開機順序控制 |
| **DSP1010** | Record Log Profile | 系統事件紀錄 |
| **DSP1076** | KVM Redirection Profile | 遠端鍵盤／影像／滑鼠 |
| **DSP1077** | USB Redirection Profile | 遠端 USB 重新導向 |
| **DSP1086** | Media Redirection Profile | 遠端媒體重新導向 |
| **DSP1018** | Service Processor Profile | BMC／服務處理器管理 |

## 結論

DMTF CIM 具備描述一台完整筆記型電腦的能力：

1. **本體層次**：`CIM_UnitaryComputerSystem.Dedicated = 33` 明確標示為 Laptop，`CIM_Chassis.ChassisPackageType = 9/10` 進一步指定外殼型態。
2. **元件層次**：CPU、記憶體（邏輯與物理 DIMM）、儲存、內建顯示器（`CIM_FlatPanel.BuiltIn = True`）、鍵盤、觸控板、電池（包含化學類型、設計容量、目前電量、健康度、充電次數等細緻屬性）、無線／有線網路、BIOS/UEFI、作業系統、散熱風扇、溫度電壓感應器——均有對應的 CIM 類別與豐富的屬性。
3. **規範層次**：DASH 規範為筆記型電腦管理提供了完整框架；DSP1058（Base Desktop and Mobile Profile）為其核心 Profile，另有二十餘個相關 Profile 分層管理各子系統。
4. **實作層次**：Intel AMT、Microsoft WMI、SBLIM、OpenLMI 等系統已在現實中將 CIM 用於筆記型電腦的管理。

**結論：是，DMTF CIM 完全能夠描述一台筆記型電腦。**

---

[^cim-what]: DMTF. (n.d.). CIM — Common Information Model. Retrieved 2026-10-01, from https://www.dmtf.org/standards/cim

[^cim-wiki]: Wikipedia. (n.d.). Common Information Model (computing). Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/Common_Information_Model_(computing)

[^cim-impl]: Microsoft. (n.d.). CIM Classes / Windows Management Instrumentation. Retrieved 2026-10-01, from https://learn.microsoft.com/en-us/windows/win32/wmisdk/cimclas

[^dedicated]: DMTF. (n.d.). CIM_ComputerSystem — Dedicated property enumeration (v2.55.0). Value 33 = "Laptop". Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.55.0/CIM_ComputerSystem.html

[^chassis]: DMTF. (n.d.). CIM_Chassis — ChassisPackageType enumeration (v2.41.0). Value 9 = "LapTop", 10 = "Notebook". Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.41.0/CIM_Chassis.html

[^processor]: DMTF. (n.d.). CIM_Processor (v2.53.0). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.53.0/CIM_Processor.html

[^memory]: DMTF. (n.d.). CIM_Memory (v2.53.0). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.53.0/CIM_Memory.html

[^physical-memory]: DMTF. (n.d.). CIM_PhysicalMemory (v2.55.0). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.55.0/CIM_PhysicalMemory.html

[^diskdrive]: Microsoft. (n.d.). CIM_DiskDrive. Retrieved 2026-10-01, from https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/cim-diskdrive

[^display]: DMTF. (n.d.). CIM_Display (v2.37.0+). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.37.0+/CIM_Display.html

[^flatpanel]: DMTF. (n.d.). CIM_FlatPanel (v2.37.0+). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.37.0+/CIM_FlatPanel.html

[^keyboard]: Microsoft. (n.d.). CIM_Keyboard. Retrieved 2026-10-01, from https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/cim-keyboard

[^pointing]: Microsoft. (n.d.). CIM_PointingDevice. Retrieved 2026-10-01, from https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/cim-pointingdevice

[^battery]: DMTF. (n.d.). CIM_Battery (v2.45.0+). Retrieved 2026-10-01, from https://schemas.dmtf.org/wbem/cim-html/2.45.0+/CIM_Battery.html

[^wifi]: DMTF. (n.d.). Wi-Fi Port Profile (DSP1088). Retrieved 2026-10-01, from https://www.dmtf.org/standards/profiles

[^psu]: DMTF. (n.d.). Power Supply Profile (DSP1015). Retrieved 2026-10-01, from https://www.dmtf.org/standards/profiles

[^fan]: DMTF. (n.d.). Fan Profile (DSP1013). Retrieved 2026-10-01, from https://www.dmtf.org/standards/profiles

[^sensor]: DMTF. (n.d.). Sensors Profile (DSP1009). Retrieved 2026-10-01, from https://www.dmtf.org/standards/profiles

[^dash]: DMTF. (n.d.). DASH — Desktop and mobile Architecture for System Hardware. Retrieved 2026-10-01, from https://www.dmtf.org/standards/dash

[^profiles]: DMTF. (n.d.). DMTF Profiles. Retrieved 2026-10-01, from https://www.dmtf.org/standards/profiles