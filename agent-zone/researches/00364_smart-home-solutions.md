# 智慧居家方案總覽

## 摘要

智慧居家（Smart Home）領域方案繁多，涵蓋開源自架平台、商用雲端生態系、硬體裝置、以及通訊協定等層面。本報告綜整各主要方案，比較其架構、優劣勢與適用場景，以提供完整的技術評估參考。

## 1. 開源／自架平台（Open Source / Self-hosted）

### 1.1 Home Assistant

- **授權**：Apache 2.0 開源；由 Open Home Foundation（非營利）管理，Nabu Casa, Inc. 提供付費雲端服務[^ha-about]。
- **架構**：Python 後端，集中式伺服器架構。提供 Home Assistant OS（嵌入式 Linux）、Container（Docker）、Core 等安裝方式[^ha-install]。
- **本地／雲端**：預設在地端運作，所有資料停留於本地網路。Nabu Casa 雲端服務提供安全遠端存取、語音助理整合，為選用項目[^ha-cloud]。
- **整合數量**：3,100+ 整合（integrations），涵蓋 1,000+ 品牌，為目前最多之開源平台[^ha-integrations]。
- **自動化**：以 Trigger → Condition → Action 模型搭配視覺化編輯器；支援 Blueprints（社群預製範本）、Scripts、Scenes[^ha-automation]。
- **儀表板**：拖放式視覺編輯器，支援 Section View、Masonry View，以及豐富的卡片元件[^ha-dashboards]。
- **語音助理**：內建 Assist 功能，支援在地端語音處理、Wake Words、50+ 語言[^ha-voice]。
- **Add-ons**：僅限 HAOS 安裝，提供 Mosquitto（MQTT Broker）、Node-RED、ESPHome、Zigbee2MQTT、Samba、InfluxDB、Grafana 等[^ha-addons]。
- **硬體需求**：Raspberry Pi 4/5、ODROID、Intel NUC、x86/ARM PC、NAS。官方硬體包含 Home Assistant Green（隨插即用）、Yellow、SkyConnect USB（Zigbee/Thread）[^ha-install]。
- **優點**：最大整合資料庫、最活躍社群、易用的自動化編輯器、優質儀表板、內建語音、快速開發節奏（每月發行）。
- **缺點**：大型佈署可能資源吃重、每月更新可能擾動、YAML 進階設定複雜度高。
- **適用對象**：從初學者（搭配 Green 硬體）到進階 DIY 玩家皆適合。

### 1.2 openHAB

- **授權**：Eclipse Public License 開源；由 openHAB Foundation（非營利）開發[^ohab-about]。
- **架構**：Java 為基礎，建構於 Apache Karaf（OSGi container）與 Eclipse Equinox 之上。支援 Linux、macOS、Windows、Raspberry Pi（openHABian）、Docker、Synology NAS[^ohab-docs]。
- **核心概念**：Things（裝置）、Channels（邏輯連結）、Items（狀態）、Bindings（連接通訊協定）、Rules（自動化邏輯）、Pages（使用者介面）[^ohab-concepts]。
- **整合數量**：400+ Bindings，支援 3,000+ 裝置類型[^ohab-addons]。
- **自動化**：When → If → Then（Trigger → Condition → Action）模型；支援 Rules DSL、JavaScript、JRuby、Python、Groovy、Blockly 多種腳本語言[^ohab-rules]。
- **使用者介面**：Main UI（現代 Web UI）、HABPanel（儀表板）、HABot（手機最佳化）、Basic UI（輕量備用）[^ohab-docs]。
- **與 Home Assistant 比較**：
  - Java 核心，架構更穩定但學習曲線更陡
  - 整合數量較少（400+ v.s. 3,100+）
  - 開發節奏較慢但更穩定
  - 社群較小但知識密度高
- **適用對象**：Java 開發者、追求最大靈活度與穩定性的使用者、複雜／特殊化佈署場景。

### 1.3 ESPHome

- **授權**：Apache 2.0 開源；Open Home Foundation 專案，與 Home Assistant 深度整合[^esphome]。
- **運作方式**：以 YAML 描述硬體配置 → 編譯韌體 → 無線（OTA）燒錄至 ESP32/ESP8266 等微控制器 → 裝置自動出現於 Home Assistant。
- **支援硬體**：ESP32、ESP8266、RP2040/RP2350、BK72xx 等；可於桌機模擬測試[^esphome-components]。
- **元件數量**：數百種內建元件，包含 sensors（溫度、濕度、壓力、氣體、動作、距離）、switchs、lights、displays、climate、covers、fans、voice assistants、Bluetooth proxies 等。
- **優點**：不需寫 C/C++（YAML 足夠）、OTA 更新、與 HA 整合極佳、低成本（ESP32 約 $3-5 USD）、龐大社群裝置資料庫。
- **缺點**：需要 DIY 硬體組裝、限於 ESP 系列微控制器、WiFi 為主（電池裝置需最佳化）。
- **適用對象**：DIY 愛好者、自訂感測器／開關需求、將傳統裝置智慧化。

### 1.4 Node-RED

- **授權**：Apache 2.0 開源；原 IBM 開發，現屬 OpenJS Foundation[^nodered]。
- **架構**：Node.js 事件驅動、流程式視覺程式設計工具。5,000+ 社群節點與流程。
- **家庭自動化應用**：常用於 Home Assistant 旁的規則引擎，處理複雜邏輯。透過 MQTT、HTTP、Home Assistant 節點與裝置互動。
- **優點**：極度靈活、視覺流程直觀、適合複雜多步驟自動化、與 HA 互補。
- **缺點**：非完整家庭自動化平台（需搭配 HA 或其他系統）、儀表板功能基本、流程易變 spaghetti、無自動發現。
- **適用對象**：進階使用者、HA 內建自動化編輯器無法滿足、需要複雜條件邏輯、跨系統整合。

### 1.5 其他值得注意的開源平台

- **Domoticz**：輕量級開源系統，支援 Z-Wave、Zigbee、MQTT，適合 Raspberry Pi 執行[^domoticz]。
- **Jeedom**：法國起源，在法語區流行。
- **FHEM**：Perl 為基礎，在德語區受歡迎。
- **ioBroker**：JavaScript 為基礎，德國起源的模組化平台。
- **Homebridge**：模擬 Apple HomeKit API，使非 HomeKit 裝置能與 Apple 生態系整合。
- **LinuxMCE**：全屋自動化與媒體中心整合。
- **MisterHouse**：Perl 架構，歷史悠久的開源平台之一。

### 1.6 通訊協定橋接工具

- **Zigbee2MQTT**：以 $20 等級的 CC2531/CC2652 USB 作為 Zigbee Coordinator，透過 MQTT 將 Zigbee 裝置暴露給任何 MQTT 消費者[^z2m]。
- **Mosquitto**：Eclipse 基金會的 MQTT Broker，為開源家庭自動化中最流行的訊息佇列[^mosquitto]。
- **Z-Wave JS**：Home Assistant 官方推薦的 Z-Wave 整合方案。

## 2. 商用生態系（Commercial Ecosystems）

### 2.1 Apple Home / HomeKit

- **類型**：封閉、專有協定（HomeKit Accessory Protocol）[^apple-homekit]。
- **本地／雲端**：在地端優先，核心功能無需網際網路。遠端存取需 Apple 裝置生態。
- **硬體需求**：需 Apple 裝置作為 Hub（HomePod、Apple TV、iPad 等）。HomePod（第 2 代）與 HomePod mini 支援 Matter Controller 與 Thread Border Router。
- **協定支援**：WiFi、Thread、Zigbee（經由 Bridge）、Matter。
- **優點**：在地端優先、隱私較佳、Apple 生態整合流暢。
- **缺點**：僅限 Apple 生態、裝置選擇較少、進階自動化能力有限。

### 2.2 Google Home / Google Nest

- **類型**：封閉、專有平台[^google-home]。
- **本地／雲端**：混合模式 — 部分指令在地端處理，但語音依賴雲端。
- **硬體**：Nest 系列智慧音箱／螢幕、Google TV Streamer、Nest Wifi Pro（內建 Thread Border Router）。
- **協定支援**：WiFi、Thread、Matter。
- **優點**：語音辨識能力強、Google 生態整合、Matter 支援完整。
- **缺點**：高度雲端依賴、隱私考量、進階自動化有限。

### 2.3 Amazon Alexa

- **類型**：封閉、專有平台[^alexa]。
- **本地／雲端**：高度依賴雲端（語音處理、Alexa Skills）。部分 Matter 裝置可在地端控制。
- **硬體**：Amazon Echo 系列（第 4 代 Echo、Echo Studio 支援 Thread/Matter）。
- **協定支援**：WiFi、Zigbee（部分 Echo 裝置內建）、Matter、Thread（經由相容 Eero 路由器）。
- **優點**：最大第三方裝置生態、Skills 豐富、Marketplace 成熟。
- **缺點**：雲端依賴高、隱私爭議多。

### 2.4 Samsung SmartThings

- **類型**：封閉、專有（原為 Kickstarter 新創，2014 年被 Samsung 收購）[^smartthings]。
- **本地／雲端**：傳統上雲端依賴；新架構與 Matter 支援改善在地端運作能力。
- **硬體**：SmartThings Hub v3、Station、Aeotec Hubs；亦內建於 Samsung 智慧電視、冰箱、螢幕。
- **協定支援**：Zigbee、Z-Wave、WiFi、Matter、Thread。
- **使用者規模**：全球 4.3 億以上使用者。
- **優點**：跨裝置品牌整合、430M+ 使用者生態、Z-Wave 原生支援。
- **缺點**：仍有一定雲端依賴、平台轉型中（Groovy → Edge）。

### 2.5 Philips Hue

- **類型**：封閉產品生態（Signify / 原 Philips Lighting）[^hue]。
- **本地／雲端**：核心功能在地端（經由 Hue Bridge）；遠端控制與語音需要雲端。
- **協定**：Zigbee Light Link（ZLL）/ Zigbee 3.0、Bluetooth（部分燈泡）。Hue Bridge Pro（2025）新增 Matter + WiFi。
- **容量**：一組 Bridge 支援 50-150 盞燈；不經 Bridge 最多 10 盞（via Bluetooth）。
- **優點**：燈產品品質與穩定性極佳、生態成熟、跨平台整合好。
- **缺點**：需 Bridge 才能發揮完整功能、價格偏高、僅限燈產品。

### 2.6 IKEA Home Smart（Dirigera）

- **類型**：封閉產品生態[^ikea]。
- **協定**：Zigbee 為主；Dirigera Hub 整合 Alexa、Google Home、Apple HomeKit。
- **產品**：燈、窗簾、感測器等。
- **優點**：價格親民、Nordic 設計、基本功能完整。
- **缺點**：產品線有限、非完整家庭自動化平台。

## 3. 中國／台灣市場相關方案

### 3.1 Xiaomi Smart Home / MIJIA（米家）

- **類型**：封閉、專有平台[^xiaomi]。
- **本地／雲端**：高度依賴雲端（中國伺服器）。小米閘道器（Gateway）可提供在地端部分自動化。
- **協定**：WiFi、Bluetooth、Zigbee（經由小米／Aqara 閘道器）。
- **生態規模**：200+ 合作夥伴品牌（Roborock、Aqara、Yeelight 等），Xiaomi HyperOS 生態系整合。
- **注意事項**：
  - 裝置常需連接中國雲端伺服器，海外使用可能出現延遲
  - 非小米生態系的第三方整合（Home Assistant 的 Xiaomi/Aqara integration）可用於在地端控制
  - Aqara 品牌裝置在國際市場支援較佳（Apple HomeKit、Matter）
- **優點**：價格低廉、產品種類極廣、生態龐大。
- **缺點**：雲端依賴高、海外使用體驗不佳、隱私風險。

### 3.2 Tuya Smart（涂鴉）

- **類型**：封閉的 white-label IoT 平台[^tuya]。
- **本地／雲端**：雲端優先。近年新產品支援部分在地端處理（Local Tuya）。
- **裝置生態**：數百個第三方品牌共用 Tuya 後端（Smart Life app 為消費者介面）。
- **協定**：WiFi、Bluetooth、Zigbee。
- **開源替代**：
  - 部分 Tuya 裝置可刷 Tasmota、ESPHome、OpenBK 等開源韌體以獲得在地端控制
  - Home Assistant 有 "Tuya Local" integration 可整合部分裝置
- **優點**：裝置種類極多、價格便宜、app 整合統一。
- **缺點**：雲端依賴、廠商不予支援後裝置變磚風險、潛在隱私問題。

### 3.3 Aqara（小米生態系衍生）

- **類型**：封閉硬體生態，但跨平台整合極佳[^aqara]。
- **本地／雲端**：支援 Apple HomeKit、Alexa、Google Home。Aqara Hub M2/M1S 支援在地端自動化執行。
- **協定**：Zigbee 3.0 為主。
- **Home Assistant 整合**：經由 Zigbee2MQTT 或 ZHA 可直接使用 Aqara 感測器，不需 Aqara Hub。
- **優點**：感測器價格極低（$10-30 USD）、國際市場支援好、HomeKit/Matter 支援佳、開源生態整合流暢。
- **缺點**：部分功能需 Aqara Hub、雲端整合仍存在。

### 3.4 中國語音助理平台

- **Alibaba Tmall Genie（天猫精靈）**：Alibaba 雲端 IoT 平台，搭配天猫精靈智慧音箱[^alibaba]。
- **Baidu DuerOS（百度小度）**：百度語音助理平台[^baidu]。
- **Tencent Xiaowei（騰訊小微）**：騰訊語音助理 IoT 平台[^tencent]。
- **ASUS Smart Home**：與 ASUS 路由器整合，支援 IFTTT、Alexa、Google Home，功能有限。

## 4. 通訊協定比較

| 協定 | 類型 | 頻段 | 拓撲 | 功耗 | 速率 | 開放性 |
|------|------|------|------|------|------|--------|
| **MQTT** | 應用層（TCP/IP） | LAN/WAN | Publish/Subscribe Broker | 極低 | 快 | Open (OASIS/ISO) |
| **Zigbee** | Mesh RF | 2.4 GHz | Mesh | 極低 | 250 kbps | IEEE 802.15.4 |
| **Z-Wave** | Mesh RF | 800-900 MHz (sub-GHz) | Mesh | 極低 | 100 kbps | 專有但標準化 |
| **Matter** | 應用層（IP） | N/A（over WiFi/Thread） | Star/IP-based | 視傳輸層 | 快 | Apache 2.0（需認證） |
| **Thread** | Mesh RF (IPv6) | 2.4 GHz | Mesh (IPv6) | 極低 | 250 kbps | Open |
| **WiFi** | Star | 2.4/5/6 GHz | Star | 高 | 54 Mbps+ | IEEE 802.11 |
| **Bluetooth/BLE** | Point-to-Point | 2.4 GHz | Star/Scatter | 極低 | 125 kbps-2 Mbps | Open |
| **KNX** | 有線/RF/IP | N/A（TP 9600 bps） | Line/Tree/Star | 極低 | 低 | ISO/IEC 14543 |

### 協定特性說明

- **MQTT**：開源家庭自動化的「黏合劑」。Publish/Subscribe 模型，極低 overhead（最小封包 2 bytes），Port 1883（未加密）/8883（TLS）。
- **Zigbee**：最普及的低功耗 Mesh 協定。用於 Hue、Aqara、IKEA、SmartThings 等。2.4 GHz 頻段可能受 WiFi 干擾。Zigbee 3.0 統一了先前紛雜的 Profile。
- **Z-Wave**：Sub-GHz 頻段（800-900 MHz），較 Zigbee 干擾少。專門認證確保互通性。在智慧鎖領域尤其強勢。
- **Thread**：基於 IPv6/6LoWPAN 的 IP Mesh 協定。設計為 Matter 的實體傳輸層。自我修復 Mesh。
- **WiFi**：直接連網的 IoT 裝置常用。功耗較高、不需 Hub。多數 Tuya/Smart Life 裝置使用 WiFi。
- **BLE**：主要用於配對設定（Hue、Xiaomi）或近接自動化。
- **KNX**：企業級建築自動化標準。有線 TP（9600 bps）、RF、IP 三種形式。需專業安裝與 ETS 軟體配置。ISO/IEC 14543 國際標準。用於高階住宅與商業建築。

## 5. 方案比較

| 維度 | 雲端依賴型 | 在地端型 | 商用封閉型 | 開源型 |
|------|-----------|---------|-----------|--------|
| **隱私** | 資料送至供應商伺服器 | 留於本地網路 | 因供應商而異 | 預設在地端 |
| **設定難度** | 容易（掃 QR Code） | 中等至困難（YAML） | 容易（app） | 中等（需基本技術） |
| **靈活度** | 限於供應商功能 | 無限（自訂腳本） | 限於 App 功能 | 無限 |
| **可靠度** | 需網際網路 | 完全離線可用 | 依賴供應商伺服器 | 完全離線可用 |
| **成本** | 免費／低價入門，進階需訂閱 | 軟體免費，硬體成本 | 中等（Hub+裝置） | 軟體免費，硬體成本 |
| **安全性** | 供應商控管 | 使用者自行控管 | 供應商控管 | 自行管理 |
| **互通性** | 封閉花園（Apple/Google/Amazon） | 通用（MQTT/REST） | 品牌生態系內 | 最大（所有協定） |
| **學習曲線** | 低（消費者友善） | 高 | 低 | 中高 |

## 6. Matter 協定

### 概要

Matter（原 Project Connected Home over IP, CHIP）由 Apple、Amazon、Google、Samsung 與 Connectivity Standards Alliance（CSA）共同推出，目的在解決智慧家庭生態系碎片化的互通性問題。2022 年 10 月發布 1.0，最新版本為 2026 年 6 月的 **1.6**[^matter]。

### 技術設計

- **IP 基礎**：運作於現有網路之上，非新無線協定
- **傳輸層**：WiFi、Ethernet、**Thread**（低功耗 mesh）
- **IPv6 定址**：採用 mDNS 進行裝置發現
- **在地端控制**：核心功能無需網際網路
- **SDK**：Apache 2.0 開源；商業產品需認證（CSA 會員＋費用）

### 版本演進

| 版本 | 新增裝置類型 |
|------|------------|
| 1.0 (2022) | 燈、插座、開關、門鎖、 thermostat、窗簾、感測器、電視 |
| 1.2 (2023) | 冰箱、冷氣、洗碗機、洗衣機、掃地機器人、煙霧警報、空氣清淨器 |
| 1.3 (2024) | 水／能源管理、烤箱、微波、爐具、場景 |
| 1.4 (2024) | 太陽能／電池、路由器、熱泵、電車充電器 |
| 1.5 (2025) | **攝影機**、土壤濕度感測器、強化工業能源管理 |
| 1.6 (2026) | NFC 配對、Joint Fabric（跨生態系分享）、Product Security 1.1（歐盟 CRA 合規） |

### 支援生態系

- **Amazon**：Echo（4th gen）、Echo Dot、Echo Hub、eero 路由器
- **Apple**：HomePod（2nd gen）、HomePod mini
- **Google**：Nest Hub（2nd gen+）、Nest Wifi Pro
- **Samsung SmartThings**：Hub v3、Station、Aeotec Hubs
- **Home Assistant**：完整 Matter Controller 支援（經由 Matter integration）

### 相容裝置品牌

Philips Hue（Bridge）、IKEA（DIRIGERA Hub）、Eve（Thread 感測器）、Nanoleaf、LIFX、TP-Link、Aqara（Hub）等。CSA 維護官方 DCL（Distributed Compliance Ledger）資料庫。

### 優點

- 統一互通標準，單一裝置可在多個生態系運作（Multi-admin / Joint Fabric）
- 在地端控制強制（核心功能不需雲端）
- 開放 SDK、所有主要玩家支援
- Thread 支援低功耗電池裝置

### 限制與批評

- 功能擴展緩慢（1.0 僅涵蓋基本裝置類型）
- 認證費用（CSA 會員＋每產品費用）對小型廠商為負擔
- 仍需 Matter Controller + 適當網路基礎設施
- 部分廠商疊加專有功能造成碎片化
- 複雜自動化不在 Matter 範疇內

## 7. 與開源相容的硬體

| 使用場景 | 推薦硬體 |
|---------|---------|
| 智慧開關／繼電器 | Shelly（WiFi，內建 HA 整合）、Sonoff（WiFi，可刷 Tasmota/ESPHome） |
| 感測器（動作、門窗、溫度） | Aqara（Zigbee，最低價）、Shelly BLU（BLE） |
| 燈光 | IKEA TRÅDFRI（預算）、Philips Hue（高階） |
| DIY 自訂感測器 | ESP32 + ESPHome |
| 長距離 Mesh | Z-Wave 裝置 |
| Bluetooth Proxy | Shelly Gen2+、ESP32 + ESPHome |

### 推薦品牌簡介

- **Sonoff（ITEAD）**：ESP8266/ESP32 基礎，可刷 Tasmota/ESPHome 開源韌體，價格 $5-20 USD[^sonoff]。
- **Aqara（小米系）**：Zigbee 感測器價格極低（$10-30 USD），透過 Zigbee2MQTT 與 HA 整合極好[^aqara]。
- **Shelly（Allterco）**：WiFi 繼電器，原生 HA 整合（CoIoT/RPC/WebSocket）、在地端控制、OTA 更新，價格 $15-50 USD[^shelly]。
- **Philips Hue**：高品質 Zigbee 燈光，可經由 Bridge 或 Zigbee2MQTT 整合。
- **IKEA TRÅDFRI**：預算 Zigbee 燈光，可跳過 IKEA Gateway 直接整合。

## 8. 總結

### 選擇建議

| 如果你… | 選擇… |
|---------|--------|
| 初學者，希望隨插即用 | **Home Assistant Green** + Sonoff/Shelly/Aqara 基本裝置 |
| 追求最大控制力，Java 開發者 | **openHAB** |
| 想 DIY 自訂感測器 | **ESPHome**（搭配 ESP32 開發板） |
| 需要複雜邏輯（HA 編輯器不夠用） | **Node-RED**（搭配 Home Assistant 使用） |
| 隱私為首要考量 | 任何自架方案（資料均停留於本地） |
| 想避免供應商鎖定 | 選用 **Matter + Thread** 認證裝置 |
| 預算有限，大量感測器需求 | Aqara（Zigbee）＋ Zigbee2MQTT ＋ Home Assistant |
| 語音助理為必要功能 | Home Assistant + Nabu Casa（Alexa/Google Home 整合）或 Apple HomeKit 生態 |

### 趨勢觀察

- **Matter** 正在成為智慧家庭的統一互通標準，但 2026 年的現實中，它仍補充而非取代既有協定（Zigbee、Z-Wave、MQTT/WiFi）。
- **在地端優先（Local-first）** 的趨勢明確：從 Google/Apple/Amazon 到 Home Assistant，都在強調在地端控制能力。
- **Home Assistant** 已成為開源智慧家庭的事實標準（2024 GitHub 貢獻者第一的開源專案）。
- **Thread** 做為 Matter 的實體層，正逐漸內建於新世代裝置與路由器。
- 中國生態系（Xiaomi、Tuya）價格極具競爭力，但雲端依賴與海外使用限制為主要痛點；搭配開源韌體或 Zigbee 協定可有效繞過。

## 參考資料

[^ha-about]: Home Assistant. (n.d.). Home Assistant – About. Retrieved 2026-10-03, from https://www.home-assistant.io/
[^ha-install]: Home Assistant. (n.d.). Installation. Retrieved 2026-10-03, from https://www.home-assistant.io/installation/
[^ha-cloud]: Home Assistant. (2026). Big Tech ruined the cloud, so we're renaming ours. Retrieved 2026-10-03, from https://www.home-assistant.io/blog/2026/10/02/big-tech-ruined-the-cloud-so-were-renaming-ours/
[^ha-integrations]: Home Assistant. (n.d.). Integrations. Retrieved 2026-10-03, from https://www.home-assistant.io/integrations/
[^ha-automation]: Home Assistant. (n.d.). Automation Basics. Retrieved 2026-10-03, from https://www.home-assistant.io/docs/automation/basics/
[^ha-dashboards]: Home Assistant. (n.d.). Dashboards. Retrieved 2026-10-03, from https://www.home-assistant.io/dashboards/
[^ha-voice]: Home Assistant. (n.d.). Voice Control. Retrieved 2026-10-03, from https://www.home-assistant.io/voice_control/
[^ha-addons]: Home Assistant. (n.d.). Add-ons. Retrieved 2026-10-03, from https://www.home-assistant.io/addons/
[^ohab-about]: openHAB Foundation. (n.d.). openHAB – About. Retrieved 2026-10-03, from https://www.openhab.org/
[^ohab-docs]: openHAB Foundation. (n.d.). openHAB Documentation. Retrieved 2026-10-03, from https://www.openhab.org/docs/
[^ohab-concepts]: openHAB Foundation. (n.d.). Concepts. Retrieved 2026-10-03, from https://www.openhab.org/docs/concepts/
[^ohab-addons]: openHAB Foundation. (n.d.). Add-ons. Retrieved 2026-10-03, from https://www.openhab.org/addons/
[^ohab-rules]: openHAB Foundation. (n.d.). Rules. Retrieved 2026-10-03, from https://www.openhab.org/docs/configuration/rules/
[^esphome]: ESPHome. (n.d.). ESPHome. Retrieved 2026-10-03, from https://esphome.io/
[^esphome-components]: ESPHome. (n.d.). Components. Retrieved 2026-10-03, from https://esphome.io/components/
[^nodered]: Node-RED. (n.d.). Node-RED – Low-code programming for event-driven applications. Retrieved 2026-10-03, from https://nodered.org/
[^domoticz]: Domoticz. (n.d.). Domoticz – Home Automation. Retrieved 2026-10-03, from https://www.domoticz.com/
[^z2m]: Zigbee2MQTT. (n.d.). Zigbee2MQTT – Bridging Zigbee to MQTT. Retrieved 2026-10-03, from https://www.zigbee2mqtt.io/
[^mosquitto]: Eclipse Foundation. (n.d.). Eclipse Mosquitto – MQTT Broker. Retrieved 2026-10-03, from https://mosquitto.org/
[^apple-homekit]: Wikipedia. (n.d.). Apple HomeKit. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Apple_HomeKit
[^google-home]: Wikipedia. (n.d.). Google Home (platform). Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Google_Home_(platform)
[^alexa]: Wikipedia. (n.d.). Amazon Alexa. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Amazon_Alexa
[^smartthings]: Wikipedia. (n.d.). SmartThings. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/SmartThings
[^hue]: Wikipedia. (n.d.). Philips Hue. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Philips_Hue
[^ikea]: Wikipedia. (n.d.). IKEA Home Smart. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/IKEA_Home_smart
[^xiaomi]: Wikipedia. (n.d.). Xiaomi Smart Home. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Xiaomi_Smart_Home
[^tuya]: Wikipedia. (n.d.). Tuya Smart. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Tuya_Smart
[^aqara]: Aqara. (n.d.). Aqara – Smart Home. Retrieved 2026-10-03, from https://www.aqara.com/
[^alibaba]: Alibaba Group. (n.d.). Tmall Genie. Retrieved 2026-10-03, from https://www.alibaba.com/
[^baidu]: Baidu. (n.d.). DuerOS. Retrieved 2026-10-03, from https://dueros.baidu.com/
[^tencent]: Tencent. (n.d.). Tencent Xiaowei. Retrieved 2026-10-03, from https://xiaowei.tencent.com/
[^matter]: Wikipedia. (n.d.). Matter (standard). Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Matter_(standard)
[^sonoff]: ITEAD. (n.d.). Sonoff – Smart Home. Retrieved 2026-10-03, from https://sonoff.itead.cc/
[^shelly]: Shelly. (n.d.). Shelly Smart Home. Retrieved 2026-10-03, from https://www.shelly.com/