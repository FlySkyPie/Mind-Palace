# FOSS 智慧居家方案綜合調查

## 概述

本報告調查 2026 年目前活躍的 FOSS（自由及開源軟體）智慧居家方案，涵蓋核心平台、語音助理、硬體韌體、通訊協定四大層面，並提供比較與建議。

---

## 一、核心平台

### 1. Home Assistant

- **網站：** home-assistant.io
- **狀態：** 目前最活躍、社群最大的 FOSS 智慧居家平台。2025 年 GitHub Octoverse 評為貢獻者最多的開源專案。全球超過 270 萬戶使用。[^ha-about]
- **技術棧：** Python 3.x 核心，模組化架構
- **整合數量：** 1,500+ 整合，涵蓋 1,000+ 品牌 [^ha-integrations]
- **UI：** Lovelace 儀表板，拖曳編輯，支援 iOS/Android/Apple Watch 原生應用
- **自動化：** YAML 或視覺化編輯器，另可搭配 Node-RED
- **語音控制：** 內建 **Assist** — 完全本地化、支援喚醒詞、70+ 語言，可串接本地 LLM 增加 AI 人格 [^ha-voice]
- **治理：** Open Home Foundation（非營利組織）[^ohf]
- **硬體：** Home Assistant Green（入門主機）、Connect ZBT-2（Zigbee/Thread）、Voice Preview Edition
- **安裝難度：** 中等。使用 Green 硬體則隨插即用

### 2. openHAB

- **網站：** openhab.org
- **狀態：** 成熟穩定，第二大 FOSS 平台。2013 年起開發 [^openhab-about]
- **技術棧：** **Java**（Apache Karaf/OSGi 執行環境），需 Java SE 21
- **整合數量：** 400+ 擴充，支援 3,000+ 裝置 [^openhab-addons]
- **UI：** 4.x Main UI 功能完整，但美觀度不及 Home Assistant
- **治理：** openHAB Foundation（非營利組織）
- **社群：** 2.2 萬論壇成員、24 萬則討論 [^openhab-community]
- **安裝難度：** 中高。openHABian 可簡化樹莓派安裝，但 Java/OSGi 學習曲線較陡

### 3. Domoticz

- **網站：** domoticz.com
- **狀態：** 輕量級、高度穩定。14+ 年開發歷史 [^domoticz-about]
- **技術棧：** **C++** — RAM 使用量低於 50 MB
- **整合數量：** 150+ 裝置類型
- **UI：** HTML5 自適應網頁，功能完整但設計較老
- **自動化：** dzVents（Lua）、Python 插件、Blockly、Lua 腳本
- **安裝難度：** 極低 — `curl -sSL install.domoticz.com | sudo bash` 單行指令安裝
- **特點：** 穩定性第一，無 breaking changes，可跑在 Pi Zero

### 4. 其他平台

| 平台 | 技術棧 | 特色 |
|---|---|---|
| ioBroker | Node.js | 德國社群盛行 |
| Gladys Assistant | Node.js | 法國起源，UI 乾淨 |
| Node-RED | Node.js | 視覺化流程編程，常搭配 Home Assistant 使用 |

---

## 二、語音助理 / 智慧 speakers（FOSS）

| 專案 | 狀態 | 備註 |
|---|---|---|
| **Home Assistant Assist** | **活躍 / 推薦** | 完全本地化、喚醒詞、70+ 語言、支援 ESP32 衛星裝置 |
| **Piper**（TTS） | **活躍** | Open Home Foundation 維護的神經 TTS 引擎，5.7k stars |
| **Whisper**（STT） | **活躍** | OpenAI Whisper 本地語音辨識 |
| **Mycroft** | **已封存（2024.09）** | 不再維護，已被 OpenVoiceOS 取代 |
| **OpenVoiceOS（OVOS）** | **活躍** | Mycroft 主要繼承者，支援 Raspberry Pi 映像 |
| **Neon AI（NeonCore）** | **活躍** | Mycroft 另一繼承者，多用戶支援，Docker 部署 |
| **Rhasspy** | **已封存（2025.10）** | 其 TTS 元件 Piper 由 OHF 繼承維護 |

---

## 三、硬體韌體（FOSS）

### ESPHome

- **狀態：** 極活躍，Open Home Foundation 專案 [^esphome-about]
- **用途：** 為 ESP32/ESP8266 微控制器打造自訂智慧裝置
- **特色：** 視覺化 Device Builder（無需寫程式）、OTA 無線更新、自動 HA 探索、100+ 元件
- **支援晶片：** ESP32, ESP8266, RP2040, BK72xx, nRF52 等

### Tasmota

- **狀態：** 極活躍，v15.6.0 Sylvie [^tasmota-about]
- **GitHub：** 24.8k stars，21,672 commits [^tasmota-gh]
- **用途：** 替代 ESP8266/ESP32 裝置的原廠韌體
- **特色：** MQTT v3.1.1/v5、**原生 Matter 支援**（ESP32）、150+ 感應器、Berry 腳本語言、Web Installer
- **支援晶片：** ESP8266, ESP32/32-S2/S3/C3/C5/C6

### OpenBeken（OpenBK7231T_App）

- **GitHub：** 2.3k stars，618 forks [^openbeken-gh]
- **用途：** 針對 **非 ESP 晶片**（Tuya 常用）的 Tasmota/ESPHome 替代韌體
- **特色：** Tasmota 相容指令、MQTT、OTA、HA 自動探索、800+ 裝置模板
- **支援晶片：** BK7231T/N, BL602, W600, LN882H, Realtek RTL8710B/8720C, XR809, ESP32/8266 等

---

## 四、通訊協定

| 協定 | 類型 | 說明 |
|---|---|---|
| **MQTT** | 發佈/訂閱 | Mosquitto 為核心 broker，所有平台皆支援 |
| **Zigbee** | 2.4GHz 網狀網路 | **Zigbee2MQTT**（15.7k stars）為黃金標準，無需廠商 hub |
| **Z-Wave** | Sub-GHz 網狀網路 | **Z-Wave JS**（OHF 維護），支援 500/700/800 系列 |
| **Matter** | IP 基礎（Wi-Fi/Thread） | Tasmota 原生支援，Home Assistant Matter controller |
| **Thread** | Matter 底層網狀 | Connect ZBT-2 內建 Thread border router |
| **ESP-NOW** | ESP 點對點 | Tasmota 的 TasMesh 功能 |
| **BLE** | 短距離 | ESPHome Bluetooth proxy、Tasmota BLE gateway |

---

## 五、比較總表

| 項目 | Home Assistant | openHAB | Domoticz |
|---|---|---|---|
| 語言 | Python | Java | C++ |
| RAM 使用 | ~300-500 MB | ~200-400 MB | ~50 MB |
| 整合數量 | 1,500+ | 400+ | 150+ |
| 社群規模 | 最大（270萬+用戶） | 大（2.2萬論壇成員） | 中（1.6萬成員） |
| UI 精緻度 | ★★★★★ | ★★★★ | ★★★ |
| 安裝難度 | 中等 | 中高 | 極低 |
| 內建語音 | Assist（完整本地化） | 雲端整合 | 基本 Alexa |
| 自動化能力 | 極高 | 高 | 中高 |
| 穩定度 | 月月更新，偶有 breaking | 極穩定 | 最穩定，無 breaking |

---

## 六、2026 年推薦組合

| 層面 | 建議 |
|---|---|
| **核心平台** | **Home Assistant**（社群最大、整合最多、UI 最佳、語音最完整） |
| **語音** | **Assist** + **Piper** TTS + **microWakeWord** |
| **Zigbee** | **Zigbee2MQTT** 或內建 ZHA |
| **Z-Wave** | **Z-Wave JS** |
| **DIY 裝置** | **ESPHome**（ESP32/ESP8266） |
| **閃刷商用裝置** | **Tasmota**（ESP 晶片）、**OpenBeken**（BK/Realtek 晶片） |
| **進階自動化** | Home Assistant 自動化 + **Node-RED** |
| **治理組織** | **Open Home Foundation**（HA, ESPHome, Piper, Z-Wave JS, Matter.js, Zigpy） |

### 依需求替代方案

- **最低資源：** Domoticz（Pi Zero、50 MB RAM）
- **企業級穩定：** openHAB（Java 架構、經生產驗證）
- **非 ESP Tuya 裝置：** OpenBeken（可閃刷 $2 的 Tuya 智慧插座）
- **純語音助理硬體：** OpenVoiceOS（Raspberry Pi + 麥克風陣列）
- **完全脫離雲端：** Home Assistant + ESPHome + Zigbee2MQTT

---

[^ha-about]: Home Assistant. (n.d.). Home Assistant — Awaken your home. Retrieved 2026-10-03, from https://www.home-assistant.io/
[^ha-integrations]: Home Assistant. (n.d.). Integrations. Retrieved 2026-10-03, from https://www.home-assistant.io/integrations/
[^ha-voice]: Home Assistant. (n.d.). Voice control. Retrieved 2026-10-03, from https://www.home-assistant.io/voice_control/
[^ohf]: Open Home Foundation. (n.d.). Open Home Foundation. Retrieved 2026-10-03, from https://www.openhomefoundation.org/
[^openhab-about]: openHAB. (n.d.). openHAB — Empowering the smart home. Retrieved 2026-10-03, from https://www.openhab.org/
[^openhab-addons]: openHAB. (n.d.). Add-ons. Retrieved 2026-10-03, from https://www.openhab.org/addons/
[^openhab-community]: openHAB. (n.d.). Community. Retrieved 2026-10-03, from https://www.openhab.org/community/
[^domoticz-about]: Domoticz. (n.d.). Domoticz — Free Open Source Home Automation System. Retrieved 2026-10-03, from https://www.domoticz.com/
[^esphome-about]: ESPHome. (n.d.). ESPHome — Easy ESP32/ESP8266 home automation. Retrieved 2026-10-03, from https://esphome.io/
[^tasmota-about]: Tasmota. (n.d.). Tasmota — Alternative firmware for ESP devices. Retrieved 2026-10-03, from https://tasmota.github.io/docs/
[^tasmota-gh]: arendst/Tasmota. (n.d.). GitHub repository. Retrieved 2026-10-03, from https://github.com/arendst/Tasmota
[^openbeken-gh]: openshwprojects/OpenBK7231T_App. (n.d.). GitHub repository. Retrieved 2026-10-03, from https://github.com/openshwprojects/OpenBK7231T_App