# Redfish API：描述裝置之間乙太網路拓撲的方法

## 概述

DMTF（Distributed Management Task Force）制定的 Redfish 規範是一套基於 RESTful 介面的硬體管理標準，可透過統一的 API 描述資料中心內伺服器、網路交換器與儲存裝置之間的乙太網路拓撲關係。本文說明 Redfish 中與乙太網路拓撲相關的核心資源結構、資源間的連結方式，以及 LLDP（Link Layer Discovery Protocol）鄰居資訊的呈現方法。

## 核心拓撲資源

### Fabric（網路架構）

Fabric（`#Fabric.v1_4_0`）是最高層級的網路拓撲容器，代表由交換器（Switch）、端點（Endpoint）與區域（Zone）組成的網路結構[^dmtf-fabric]。

```json
{
    "@odata.type": "#Fabric.v1_4_0.Fabric",
    "@odata.id": "/redfish/v1/Fabrics/Ethernet",
    "Id": "Ethernet",
    "Name": "乙太網路架構",
    "FabricType": "Ethernet",
    "Switches": { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Switches" },
    "Endpoints": { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Endpoints" },
    "Zones": { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Zones" }
}
```

### Switch（交換器）

交換器（`#Switch.v1_10_0`）描述 Fabric 中的交換裝置。其 `Ports` 屬性指向 Port 集合，`Links` 物件則關聯到管理的機箱（Chassis）、管理器（Manager）關聯的端點[^dmtf-switch]。

關鍵屬性：

- `SwitchType`：通訊協定類型（Ethernet、FibreChannel、PCIe、CXL 等）
- `IsManaged`：是否受管理
- `TotalSwitchWidth`：實體埠總數
- `CurrentBandwidthGbps`／`MaxBandwidthGbps`：頻寬資訊
- `Links.Endpoints`：此交換器連接的端點
- `Links.Chassis`：所屬機箱
- `Links.ManagedBy`：管理控制器

### Port（埠）

Port（`#Port.v1_19_0`）是描述拓撲的核心資源，可用於交換器或網路卡（NIC）上的實體埠。**注意：舊版的 `NetworkPort` 已棄用，應改用 `Port`**[^dmtf-port]。

Port 間的連線關係透過 `Links` 物件表達：

| Links 屬性 | 類型 | 說明 |
|---|---|---|
| `ConnectedPorts` | `Port[]` | 對端裝置的埠（v1.2+） |
| `ConnectedSwitchPorts` | `Port[]` | 對端交換器埠 |
| `ConnectedSwitches` | `Switch[]` | 對端交換器 |
| `AssociatedEndpoints` | `Endpoint[]` | 與此埠關聯的端點 |
| `Cables` | `Cable[]` | 連接至此埠的纜線（v1.5+） |
| `EthernetInterfaces` | `EthernetInterface[]` | 此埠提供的乙太網路介面（v1.7+） |

重要連結狀態屬性：

- `LinkStatus`：`LinkUp`、`Starting`、`Training`、`LinkDown`、`NoLink`
- `LinkState`：`Enabled`、`Disabled`
- `CurrentSpeedGbps`：當前協商速度
- `MaxSpeedGbps`：最大支援速度
- `ActiveWidth`：啟用的通道數

### NetworkAdapter（網路卡）

`#NetworkAdapter.v1_13_0` 描述伺服器中安裝的實體網路介面卡（NIC），包含控制器資訊、埠集合（`Ports`）與網路裝置功能集合（`NetworkDeviceFunctions`）[^dmtf-netadapter]。

### NetworkDeviceFunction（網路裝置功能）

`#NetworkDeviceFunction.v1_11_1` 代表網路卡對作業系統暴露的邏輯介面（如 PCIe 實體功能 PF 或虛擬功能 VF），包含 Ethernet 區塊中 MAC 位址、VLAN 設定、連接狀態等屬性[^dmtf-ndf]。

### EthernetInterface（乙太網路介面）

`#EthernetInterface.v1_2_14` 位於系統（`ComputerSystem`）或管理器（`Manager`）路徑下，代表 OS 層級可見的網路介面，包含 IP 位址、MAC 位址、DNS 設定等[^dmtf-ei]。

## 資源層級關係

```
Fabric (Ethernet)
├── Switches[]
│    └── Switch (ToR 交換器)
│         └── Ports[]
│              └── Port
│                   ├── Links.ConnectedPorts[] → 對端 Port
│                   ├── Links.ConnectedSwitches[] → 對端 Switch
│                   └── Links.AssociatedEndpoints[] → 端點

Chassis (伺服器機箱)
└── NetworkAdapter (NIC 卡)
     ├── Ports[] → Port (實體埠)
     │    └── Links.ConnectedSwitchPorts[] → 交換器 Port
     └── NetworkDeviceFunctions[] → 邏輯功能
          └── PhysicalPortAssignment → 對應的 Port

ComputerSystem (主機系統)
└── EthernetInterfaces[] → OS 網路介面
```

## 完整連線範例

以下範例展示一台伺服器（Server1）透過 NIC 連接至 ToR 交換器（TorSwitch1）的完整資源結構。

### 交換器埠

```json
{
    "@odata.type": "#Port.v1_19_0.Port",
    "@odata.id": "/redfish/v1/Fabrics/Ethernet/Switches/TorSwitch1/Ports/1",
    "Id": "1",
    "PortId": "1/1/1",
    "PortProtocol": "Ethernet",
    "CurrentSpeedGbps": 25,
    "LinkStatus": "LinkUp",
    "LinkState": "Enabled",
    "Ethernet": {
        "LLDPEnabled": true,
        "LLDPReceive": {
            "ChassisId": "MAC: 94:6d:ae:5c:9d:cd",
            "PortId": "MAC: 94:6d:ae:5c:9d:cd",
            "SystemName": "server1-nic",
            "SystemCapabilities": ["Station"]
        }
    },
    "Links": {
        "ConnectedPorts": [
            { "@odata.id": "/redfish/v1/Chassis/Server1/NetworkAdapters/NIC1/Ports/1" }
        ],
        "AssociatedEndpoints": [
            { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Endpoints/Server1NIC" }
        ]
    }
}
```

### 伺服器網路卡 Port

```json
{
    "@odata.type": "#Port.v1_19_0.Port",
    "@odata.id": "/redfish/v1/Chassis/Server1/NetworkAdapters/NIC1/Ports/1",
    "Id": "1",
    "PortProtocol": "Ethernet",
    "CurrentSpeedGbps": 25,
    "LinkStatus": "LinkUp",
    "Links": {
        "ConnectedSwitchPorts": [
            { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Switches/TorSwitch1/Ports/1" }
        ]
    }
}
```

### 網路裝置功能

```json
{
    "@odata.type": "#NetworkDeviceFunction.v1_11_1.NetworkDeviceFunction",
    "@odata.id": "/redfish/v1/Chassis/Server1/NetworkAdapters/NIC1/NetworkDeviceFunctions/1.1",
    "Id": "1.1",
    "NetDevFuncType": "Ethernet",
    "Ethernet": {
        "MACAddress": "94:6d:ae:5c:9d:ce",
        "VLAN": { "VLANId": 100, "VLANEnabled": true },
        "LinkStatus": "Up",
        "SpeedMbps": 25000
    },
    "PhysicalPortAssignment": {
        "@odata.id": "/redfish/v1/Chassis/Server1/NetworkAdapters/NIC1/Ports/1"
    }
}
```

### 主機乙太網路介面

```json
{
    "@odata.type": "#EthernetInterface.v1_2_14.EthernetInterface",
    "@odata.id": "/redfish/v1/Systems/Server1/EthernetInterfaces/eth0",
    "MACAddress": "94:6d:ae:5c:9d:ce",
    "SpeedMbps": 25000,
    "LinkStatus": "Up",
    "IPv4Addresses": [
        {
            "Address": "192.168.1.100",
            "SubnetMask": "255.255.255.0",
            "AddressOrigin": "DHCP",
            "Gateway": "192.168.1.1"
        }
    ]
}
```

## LLDP 鄰居資訊

LLDP（IEEE 802.1AB）資訊位於 Port 資源的 `Ethernet` 區塊中，包含接收（`LLDPReceive`）與發送（`LLDPTransmit`）兩組資料[^nvidia-lldp]。

### LLDPReceive（接收的鄰居資訊）

- `ChassisId`／`ChassisIdSubtype`：對端設備機箱識別碼
- `PortId`／`PortIdSubtype`：對端埠識別碼
- `SystemName`／`SystemDescription`：鄰居裝置名稱與描述
- `SystemCapabilities`：裝置能力（Bridge、Router、Station 等）
- `ManagementAddressIPv4`／`ManagementAddressIPv6`：管理位址
- `ManagementVlanId`：管理 VLAN

### 關於多個 LLDP 鄰居的限制

根據 Redfish 規格討論，`LLDPReceive` 並非集合（collection）類型，每個 Port 預設僅支援一個 LLDP 鄰居。若需表示多個鄰居連接至同一埠（如透過 Tap 或分光器），應將每個鄰居建模為**不同的 Port 資源**，而非在同一個 Port 下陳列多筆 `LLDPReceive`[^redfish-multi-lldp]。

## 交換器間連結（ISL）

交換器之間的乙太網路拓撲（Inter-Switch Link）透過雙向的 `ConnnectedSwitchPorts`／`ConnectedSwitches` 連結來表達：

```json
// 交換器 A 的埠
"Links": {
    "ConnectedSwitchPorts": [
        { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Switches/SwitchB/Ports/24" }
    ],
    "ConnectedSwitches": [
        { "@odata.id": "/redfish/v1/Fabrics/Ethernet/Switches/SwitchB" }
    ]
}
```

兩個交換器應各自在對應的 Port 上相互參照，以構成完整的雙向拓撲。

## Zones（區域）與 Endpoints（端點）

Zone（`#Zone`）用於將 Fabric 中的端點分組，表達 VLAN 或網路分割等隔離概念。Endpoint（`#Endpoint`）代表網路中的連線終點，可透過 `ConnectedEntities` 陣列參照至實體埠或 PCIe 功能[^dmtf-fabric]。

## 小結

Redfish 透過 Fabric → Switch → Port → Links 的階層式資源結構，搭配 LLDP 資訊與雙向連結，可完整描述資料中心的乙太網路拓撲。關鍵設計原則：

1. **使用 `Port`（而非已棄用的 `NetworkPort`）** 作為統一埠資源
2. **透過 `Port.Links` 的雙向參照**建構連線關係
3. **LLDP 資訊**嵌入 Port 的 `Ethernet` 區塊
4. **Fabric 層級**提供交換器、端點與區域的整體視圖
5. **端對端路徑**由 NetworkAdapter → Port → NetworkDeviceFunction → EthernetInterface 的層級關係構成

[^dmtf-fabric]: DMTF. (n.d.). Fabric Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/Fabric.v1_4_0.json

[^dmtf-switch]: DMTF. (n.d.). Switch Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/Switch.v1_10_0.json

[^dmtf-port]: DMTF. (n.d.). Port Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/Port.v1_19_0.json

[^dmtf-netadapter]: DMTF. (n.d.). NetworkAdapter Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/NetworkAdapter.json

[^dmtf-ndf]: DMTF. (n.d.). NetworkDeviceFunction Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/NetworkDeviceFunction.json

[^dmtf-ei]: DMTF. (n.d.). EthernetInterface Schema. Retrieved 2026-09-26, from https://redfish.dmtf.org/schemas/v1/EthernetInterface.json

[^nvidia-lldp]: NVIDIA. (2024). LLDP in Redfish — BlueField BMC Documentation. Retrieved 2026-09-26, from https://networking-docs.nvidia.com/bluefieldbmc/26.07/lldp-in-redfish

[^redfish-multi-lldp]: Redfish Forum. (2025). Representing Multiple LLDP Neighbors. Retrieved 2026-09-26, from https://redfishforum.com/thread/1143/representing-multiple-lldp-neighbors