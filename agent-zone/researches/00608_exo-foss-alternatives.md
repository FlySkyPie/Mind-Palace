# exo 的開源替代方案：分散式 LLM 推論工具的 FOSS 生態調查

本報告調查與 [exo-explore/exo](https://github.com/exo-explore/exo) 功能相近的自由與開源軟體（FOSS, Free and Open-Source Software）替代方案。exo 是一套將多台個人裝置（Mac、PC、手機、筆電）串聯成單一 AI 叢集、藉此在本機執行大型語言模型（LLM, Large Language Model）的分散式推論平台。本報告先說明 exo 的核心特性，再逐一介紹功能相近的開源專案，最後以表格比較，協助讀者依自身硬體與需求選擇合適工具。

## 一、基準專案：exo

exo 的定位是「在本機執行前沿 AI 模型」（Run frontier AI locally）[^exorepo]。其核心概念是把所有可用裝置的算力與記憶體彙整成一個叢集，讓單一裝置無法容納的大模型得以執行。

主要技術特性包括：

- **自動裝置探索**：裝置間透過 Zenoh（一套 pub/sub 訊息傳遞層）自動互相發現，無需手動設定[^exorepo]。
- **拓樸感知自動平行化**：即時建立裝置拓樸視圖（資源、網路延遲、各連線頻寬），自動決定最佳的模型切分方式[^exorepo]。
- **張量平行（Tensor Parallelism）**：將模型張量切分到多台裝置，官方基準顯示 2 台裝置有 1.8 倍加速、4 台有 3.2 倍加速[^exorepo]。
- **管道平行（Pipeline Parallelism）**：以層為單位切分，不同層於不同裝置上執行[^exorepo]。
- **Thunderbolt 5 上的 RDMA**：在 macOS 26.2+ 且支援 Thunderbolt 5 的機器上支援遠端直接記憶體存取（RDMA, Remote Direct Memory Access），可降低裝置間延遲達 99%[^exorepo]。
- **多種 API 相容性**：相容 OpenAI Chat Completions、Anthropic Claude Messages、OpenAI Responses、以及 Ollama API[^exorepo]。
- **內建儀表板與 macOS 應用程式**：提供網頁管理介面與原生 macOS 選單列應用程式[^exorepo]。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/exo-explore/exo |
| 星數 | 約 48,000 |
| 授權 | Apache-2.0 |
| 主要語言 | Python（核心）、Rust、TypeScript |
| 平台重點 | macOS / Apple Silicon 優先，Linux 目前僅 CPU |

## 二、最相近的替代方案

### 1. Petals

Petals 由 BigScience 研究工作坊開發，採用「BitTorrent 風格」的分散式架構，讓參與者各自託管模型的一小部分層，並透過公開或私人「swarm」網路協同推論[^petalsrepo]。它使用管道平行（依層切分），支援推論與微調，可執行 Llama 3.1（最高 405B）、Mixtral、Falcon、BLOOM 等模型[^petalsrepo]。與 exo 最大的差異在於：Petals 以**跨網際網路的志願運算**為設計目標，而非同一區網內的私人裝置叢集。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/bigscience-workshop/petals |
| 星數 | 約 10,700 |
| 授權 | MIT |
| 主要語言 | Python（PyTorch） |

### 2. distributed-llama

由 b4rtaz 開發，用 C++ 撰寫，將家用裝置串聯成叢集，以張量平行搭配乙太網路同步加速推論[^dllamarepo]。支援 Linux、macOS、Windows，並針對 ARM 與 x86_64 AVX2 CPU 最佳化，具實驗性 Vulkan GPU 支援，可執行 Llama 3.x、Qwen 3、DeepSeek 等模型[^dllamarepo]。與 exo 相比更輕量、更精簡，但需手動指定各節點的 IP 與埠，且節點數須為 2 的次方[^dllamarepo]。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/b4rtaz/distributed-llama |
| 星數 | 約 3,100 |
| 授權 | MIT |
| 主要語言 | C++ |

### 3. llama.cpp RPC 後端

llama.cpp 內建的 RPC（Remote Procedure Call，遠端程序呼叫）後端可將張量運算序列化後透過網路傳送到其他機器執行，讓多台機器共同分擔模型運算[^llamacpprpc]。架構上一台主機執行 `llama-cli` 或 `llama-server`，其他工作節點執行 `ggml-rpc-server`，主機以 `--split-mode layer` 將連續層區塊分配給各機器[^llamacpprpc]。支援三種傳輸：TCP（預設）、RDMA（Linux RoCEv2 / macOS Thunderbolt 5），以及透過 RDMA 的 Thunderbolt 實體鏈路[^llamacpprpc]。**重要提醒**：官方仍標示其為概念驗證階段，功能脆弱且不安全，缺乏驗證與加密，不應在開放網路或敏感環境執行[^llamacpprpc]。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/ggml-org/llama.cpp/tree/master/tools/rpc |
| 星數 | 約 132,000（整個 llama.cpp 專案） |
| 授權 | MIT |
| 主要語言 | C / C++ |

### 4. prima.cpp

prima.cpp 是 llama.cpp 的分散式實作，目標是讓 30B–70B 參數的 LLM（如 QwQ-32B、Qwen 2.5-72B、Llama 3-70B、DeepSeek R1 70B）能在異質的日常家用叢集上執行，這些裝置可涵蓋不同作業系統（macOS、Linux、Android/HarmonyOS via Termux）、不同 CPU/GPU、有限記憶體、慢速磁碟與 Wi-Fi 鏈路[^primacpp]。該專案已被 ICLR 2026 接受[^primacpp]。這是 exo 在「異質家用叢集」方向上最直接的對應方案之一。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/OpenCPIL/prima.cpp |
| 星數 | 約 115 |
| 授權 | MIT |
| 主要語言 | C++ |

## 三、其他可比較的專案

### 5. Parallax

由 Gradient 開發，是一套「完全去中心化」的推論引擎，可跨越異質節點（不限硬體配置與地理位置）建立 AI 叢集[^parallaxrepo]。採用管道平行切分模型，P2P 通訊使用 Lattica，GPU 後端用 SGLang 與 vLLM，Mac 後端用 MLX LM[^parallaxrepo]。與 exo 的差異在於：Parallax 針對不可靠的志願運算節點設計，相容性更廣但成熟度較低（v0.0.1）[^parallaxrepo]。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/GradientHQ/parallax |
| 星數 | 約 1,400 |
| 授權 | Apache-2.0 |
| 主要語言 | Python（含 Rust 元件） |

### 6. dnet

由 FirstBatch 開發，是專為 **Apple Silicon 叢集**設計的分散式 LLM 推論框架，可執行超過 32B 參數、8-bit 量化的模型[^dnetrepo]。採用管道環（pipelined-ring）架構，具自動裝置探索、Thunderbolt 偵測、異質感知的層分配求解器（基於 distilp），並提供 OpenAI 相容 API 與 Rust 終端介面[^dnetrepo]。與 exo 同樣鎖定 Apple Silicon，但規模與社群小得多。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/firstbatchxyz/dnet |
| 星數 | 約 76 |
| 授權 | Apache-2.0 |
| 主要語言 | Python（含 Rust） |

### 7. GPUStack

一套開源 GPU 叢集管理器，用於 AI 模型服務，可於地端伺服器、Kubernetes 與雲端之間編排 vLLM、SGLang、TensorRT-LLM 等推論引擎[^gpustackrepo]。具 GPU 實例調度、負載平衡、監控與 OpenAI 相容 API，支援 NVIDIA、AMD、Ascend NPU 等多種加速器[^gpustackrepo]。定位偏向企業級叢集管理，而非 exo 的家用即插即用裝置池。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/gpustack/gpustack |
| 星數 | 約 5,900 |
| 授權 | Apache-2.0 |
| 主要語言 | Python |

### 8. Kalavai

由 kalavai-net 開發，目標是彙集零散的 GPU 算力成為單一資源池[^kalavairepo]。具 seed 節點（控制平面）與 worker 節點架構，提供 vLLM、llama.cpp、SGLang、Ray 叢集等現成範本，可跨地端與多云環境[^kalavairepo]。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/kalavai-net/kalavai-client |
| 星數 | 約 223 |
| 授權 | Apache-2.0 |
| 主要語言 | TypeScript、Python |

### 9. hivemind

由 learning-at-home 開發，是 Petals 背後的底層函式庫，提供 PyTorch 的去中心化深度學習能力，以分散式雜湊表（DHT）進行對等節點探索，支援容錯反向傳播與去中心化參數平均[^hivemindrepo]。它是一套基礎函式庫而非終端使用者工具，但其去中心化技術是理解 Petals 類系統的關鍵。

| 屬性 | 值 |
|---|---|
| GitHub | https://github.com/learning-at-home/hivemind |
| 星數 | 約 2,500 |
| 授權 | MIT |
| 主要語言 | Python |

### 10. 企業級推論引擎：vLLM、SGLang、llm-d、AIBrix

以下專案並非 exo 的直接替代品，但都能進行跨節點的分散式推論，適合生產環境：

- **vLLM**（約 95,300 星，Apache-2.0）：主流開源 LLM 推論引擎，透過 NCCL 支援張量／管道／資料／專家平行，涵蓋多 GPU 與多節點，但需手動配置叢集[^vllmrepo]。
- **SGLang**（約 37,700 星，Apache-2.0）：高效能服務框架，支援跨主機的張量與專家平行，硬體涵蓋 NVIDIA、AMD、Intel、TPU、Apple Silicon、Ascend 等[^sglangrepo]。
- **llm-d**（約 5,000 星，Apache-2.0）：Kubernetes 原生的分散式推論堆疊（CNCF sandbox），由 Google Cloud、Red Hat、IBM、CoreWeave、NVIDIA 共同創立[^llmdrepo]。
- **AIBrix**（約 5,200 星，Apache-2.0）：LLM 推論的 Kubernetes 控制平面，可搭配 vLLM、SGLang、TensorRT-LLM，支援多節點分散式推論[^aibrixrepo]。

這些工具效能與生產成熟度高，但設定複雜度遠高於 exo 的自動探索設計，通常需要 Kubernetes 或明確的叢集設定。

## 四、比較總覽

| 專案 | 星數 | 授權 | 語言 | 跨裝置叢集 | 平台重點 |
|---|---|---|---|---|---|
| **exo**（基準） | 48k | Apache-2.0 | Python / Rust | 是 | Apple Silicon 優先 |
| Petals | 10.7k | MIT | Python | 是（網際網路 swarm） | 跨平台 GPU |
| distributed-llama | 3.1k | MIT | C++ | 是 | CPU 為主、跨平台 |
| llama.cpp RPC | 132k* | MIT | C/C++ | 是 | 跨平台 |
| prima.cpp | 115 | MIT | C++ | 是 | 異質家用裝置 |
| Parallax | 1.4k | Apache-2.0 | Python | 是 | 去中心化異質節點 |
| dnet | 76 | Apache-2.0 | Python | 是 | Apple Silicon |
| GPUStack | 5.9k | Apache-2.0 | Python | 是 | GPU 叢集管理 |
| Kalavai | 223 | Apache-2.0 | TypeScript | 是 | 算力彙集 |
| hivemind | 2.5k | MIT | Python | 是（底層） | 去中心化訓練函式庫 |
| vLLM | 95.3k | Apache-2.0 | Python | 是 | 生產推論引擎 |
| SGLang | 37.7k | Apache-2.0 | Python | 是 | 生產推論引擎 |
| llm-d | 5.0k | Apache-2.0 | Go / Python | 是 | Kubernetes |
| AIBrix | 5.2k | Apache-2.0 | Go / Python | 是 | Kubernetes |

\* llama.cpp RPC 為 llama.cpp 專案的子元件，星數為整個專案。

## 五、選擇建議

- **想要 Mac 之間即插即用**：exo 或 **dnet**（dnet 對 Apple Silicon 的層分配最佳化更細緻）[^dnetrepo]。
- **想要輕量、純 CPU 的家用叢集**：**distributed-llama** 或 **llama.cpp RPC**[^dllamarepo][^llamacpprpc]。
- **想要跨網際網路的志願式協作推論**：**Petals**（或底層的 **hivemind**）[^petalsrepo][^hivemindrepo]。
- **想要支援最廣泛的異質裝置（含手機）**：**prima.cpp** 或 **Parallax**[^primacpp][^parallaxrepo]。
- **生產環境、多 GPU 叢集**：**vLLM**、**SGLang**、**llm-d**、**AIBrix**，或搭配 **GPUStack** / **Kalavai** 管理[^vllmrepo][^sglangrepo][^llmdrepo][^aibrixrepo][^gpustackrepo][^kalavairepo]。

## 六、來源反思

本報告的事實主要來自各專案的官方 GitHub 儲存庫與 README，屬於一手來源，可信度高；星數為擷取當下（2026-10-23）所見數值，會隨時間變動。部分專案（如 dnet、prima.cpp、Parallax）社群規模小、仍在早期階段，其成熟度與穩定性宣稱多來自專案自我描述，尚缺乏獨立的第三方驗證，讀者應審慎看待。llama.cpp RPC 後端官方明確標示為概念驗證、不安全的狀態，這是評估時的重要限制[^llamacpprpc]。此外，exo 本身針對 Apple Silicon 深度最佳化，Linux 目前僅支援 CPU，這是跨平台使用者需納入考量的關鍵事實[^exorepo]。

## 參考文獻

[^exorepo]: exo-explore. (n.d.). *exo*. GitHub. Retrieved 2026-10-23, from https://github.com/exo-explore/exo

[^petalsrepo]: bigscience-workshop. (n.d.). *Petals*. GitHub. Retrieved 2026-10-23, from https://github.com/bigscience-workshop/petals

[^dllamarepo]: b4rtaz. (n.d.). *distributed-llama*. GitHub. Retrieved 2026-10-23, from https://github.com/b4rtaz/distributed-llama

[^llamacpprpc]: ggml-org. (n.d.). *llama.cpp RPC backend*. GitHub. Retrieved 2026-10-23, from https://github.com/ggml-org/llama.cpp/tree/master/tools/rpc

[^primacpp]: OpenCPIL. (n.d.). *prima.cpp: Scale Your Local AI Beyond One Device*. GitHub. Retrieved 2026-10-23, from https://github.com/OpenCPIL/prima.cpp

[^parallaxrepo]: GradientHQ. (n.d.). *Parallax*. GitHub. Retrieved 2026-10-23, from https://github.com/GradientHQ/parallax

[^dnetrepo]: firstbatchxyz. (n.d.). *dnet: Distributed LLM Inference for Apple Silicon Clusters*. GitHub. Retrieved 2026-10-23, from https://github.com/firstbatchxyz/dnet

[^gpustackrepo]: gpustack. (n.d.). *GPUStack*. GitHub. Retrieved 2026-10-23, from https://github.com/gpustack/gpustack

[^kalavairepo]: kalavai-net. (n.d.). *kalavai-client*. GitHub. Retrieved 2026-10-23, from https://github.com/kalavai-net/kalavai-client

[^hivemindrepo]: learning-at-home. (n.d.). *hivemind*. GitHub. Retrieved 2026-10-23, from https://github.com/learning-at-home/hivemind

[^vllmrepo]: vllm-project. (n.d.). *vLLM*. GitHub. Retrieved 2026-10-23, from https://github.com/vllm-project/vllm

[^sglangrepo]: sgl-project. (n.d.). *SGLang*. GitHub. Retrieved 2026-10-23, from https://github.com/sgl-project/sglang

[^llmdrepo]: llm-d. (n.d.). *llm-d*. GitHub. Retrieved 2026-10-23, from https://github.com/llm-d/llm-d

[^aibrixrepo]: vllm-project. (n.d.). *AIBrix*. GitHub. Retrieved 2026-10-23, from https://github.com/vllm-project/aibrix
