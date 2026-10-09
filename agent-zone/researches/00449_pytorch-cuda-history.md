# PyTorch 與 CUDA 的歷史與關聯

## 1. 緒論

PyTorch 與 CUDA 的關係是現代深度學習生態系統中最關鍵的合作之一。PyTorch 由 Meta（原 Facebook）AI Research（FAIR）於 2016 年創建，而 CUDA 則是 NVIDIA 自 2006 年以來推出的平行運算平台與 API。兩者經過近十年的共同演化，形成了深度學習領域事實上的標準組合。本報告探討 PyTorch 與 CUDA 的歷史淵源、技術整合、關鍵里程碑與未來發展方向。

## 2. PyTorch 的創建背景

### 2.1 成立時間與團隊

PyTorch 於 **2016 年** 在 Facebook AI Research（FAIR）誕生。核心開發團隊包括：

- **Soumith Chintala** — 專案負責人
- **Adam Paszke** — Autograd 架構設計師
- **Sam Gross** — 核心工程
- **Gregory Chanan** — 核心工程

PyTorch 的前身是 **Torch**，最初於 2001 年左右由 Ronan Collobert 等人在瑞士 IDIAP 研究所創建。2010-2011 年間，Torch 被重寫為 Lua 語言版本（Torch7/LuaTorch），並加入了 C/CUDA 後端，主要貢獻者包括 Ronan Collobert、Clément Farabet 與 Koray Kavukcuoglu。隨著 Python 生態系統的崛起與動態計算圖（dynamic computation graph）的需求，PyTorch 應運而生。[^wiki-pytorch] [^chintala-design]

### 2.2 首次發布

PyTorch 的首個公開 alpha 版本於 **2016 年 9 月** 發布，從第一天起就將 CUDA 支援內建於其 C++ 核心張量庫 **ATen** 中。[^chintala-design] [^wiki-pytorch]

## 3. PyTorch 如何運用 CUDA 實現 GPU 加速

PyTorch 在多個層級上深度整合 CUDA：

### 3.1 ATen — CUDA 優化的張量後端

PyTorch 的核心 C++ 張量庫 ATen 從零開始撰寫，CUDA 支援是其設計基石。它提供了 NumPy 風格的張量運算並附帶 GPU 加速。LuaTorch 時代的 TH/THC 和 THNN/THCUNN C 後端被重構為語言無關的元件，成為 PyTorch 的基礎。[^chintala-design]

### 3.2 `torch.cuda` 模組

PyTorch 提供一級公民的 `torch.cuda` 模組，讓使用者可以：

- 將張量移至 GPU：`.to('cuda')` 或 `.cuda()`
- 檢查 GPU 可用性：`torch.cuda.is_available()`
- 管理 CUDA 串流、記憶體與同步
- 透過 `torch.cuda.amp` 使用自動混合精度（Automatic Mixed Precision, AMP）

### 3.3 DistributedDataParallel（DDP）與 NCCL

`torch.nn.parallel.DistributedDataParallel` 搭配 NVIDIA 的 **NCCL**（NVIDIA Collective Communications Library）後端，成為多 GPU 訓練的標準做法。NCCL 實現了高效的 GPU 集體通訊（collective communication）。[^nvidia-nccl]

### 3.4 自動混合精度（AMP）

`torch.cuda.amp` 模組由 NVIDIA 與 Facebook 工程師合作開發，使用半精度（FP16）訓練同時保持完整網路精度，透過自動利用 NVIDIA Tensor Cores 實現最高 **3 倍效能提升**。

### 3.5 CUDA Graphs

自 PyTorch 1.10（2021 年）起引入的 CUDA Graphs API，將多個 GPU 運算打包成單一可啟動單元，大幅降低 CPU 啟動開銷。在 MLPerf 工作負載上可實現高達 **1.7 倍加速**，並成功擴展至 4000 顆以上 GPU。[^pytorch-cuda-graphs-blog] [^nvidia-cuda-graphs-docs]

### 3.6 `torch.compile`（PyTorch 2.0，2023 年）

PyTorch 2.0 的編譯堆疊（TorchDynamo → AOTAutograd → PrimTorch → TorchInductor）可從 Python 程式碼生成**優化的 CUDA kernel**，包含使用 Triton 生成自訂 kernel 的能力，在多數模型上實現 1.3 倍至 2 倍加速，且無需任何程式碼更改。[^pytorch-20]

## 4. PyTorch-CUDA 整合的關鍵里程碑

| 年份 | 里程碑 | 重要性 |
|------|--------|--------|
| ~2010 | Torch7（LuaTorch）加入 CUDA 後端 | 首個具備 CUDA 支援的主要深度學習框架 |
| 2015–2016 | 重構 TH/THC 後端為語言無關 | 使 CUDA 張量程式碼可被 PyTorch 重用 |
| 2016 年 9 月 | PyTorch alpha 發布 | 以 C++ 核心（ATen）建構，從第一天即支援 CUDA |
| 2017 年 | ONNX 發布（Facebook + Microsoft） | 讓 PyTorch 模型可透過 NVIDIA TensorRT 部署 |
| 2018 年 12 月 | **PyTorch 1.0**（與 Caffe2 合併） | 統一研究與生產路徑；TorchScript 用於部署 |
| 2019 年 | `torch.distributed` 搭配 NCCL 後端 | 多 GPU DDP 訓練成為標準 |
| 2020 年 | AMP（自動混合精度） | 透過 Tensor Cores 實現 3 倍訓練加速 |
| 2021 年（v1.10） | **CUDA Graphs API**（beta） | 約 1.7 倍 MLPerf 加速；擴展至 4000+ GPU |
| 2021 年（v1.11/1.12） | **FSDP** 上游化 | 分片訓練（sharded training）使 GPU 叢集可訓練 100B+ 參數模型 |
| 2022 年 | **PyTorch 基金會** 在 Linux 基金會下成立 | 創始成員包括 NVIDIA、Meta、AMD、AWS、Apple、Google、Microsoft |
| 2022 年（v1.12） | Apple MPS 與 AMD ROCm 後端穩定 | CUDA 不再是唯一的 GPU 後端 |
| 2023 年 3 月 | **PyTorch 2.0** 搭配 `torch.compile` | JIT 編譯器生成優化的 CUDA kernel；2 倍加速 |
| 2023 年 | `torch.cuda.CUDAGraph` 正式穩定 | 生產級 CUDA 圖執行 |
| 2024 年 | **TorchTitan** 發布 | 參考 LLM 訓練實作，具備 3D 平行化（FSDP + tensor + pipeline）在數千 GPU 上 |
| 2026 年 7 月 | **7 億次 PyPI 下載**（含 CUDA 支援） | 里程碑標誌著 PyTorch-CUDA 共同演化近十年 |
| 2025–2026 年 | **Monarch** 叢集級程式設計 | 讓 1000+ GPU 叢集感覺像單一機器 |

[^pytorch_evolution]: PyTorch 技術部落格系列。Retrieved 2026-10-09, from https://github.com/AlirezaShamsoshoara/PyTorchHistory

## 5. CUDA 如何賦能 PyTorch 的深度學習能力

### 5.1 大規模平行化

CUDA 讓 PyTorch 能存取數千顆 GPU 核心，平行執行張量運算。一次 `torch.matmul()` 呼叫會翻譯為數千個 CUDA 執行緒同時運行，矩陣運算可實現 **10–100 倍**的 CPU 加速。

### 5.2 軟體堆疊

PyTorch 運算的執行路徑為：

```
Python API → C++ (ATen) → CUDA Runtime → GPU Driver → GPU Hardware
```

CUDA 管理從參數驗證到 kernel 調度再到硬體排程的完整管線。

### 5.3 PyTorch 使用的重要 CUDA 函式庫

| 函式庫 | 用途 |
|--------|------|
| **cuBLAS** | 基礎線性代數（矩陣乘法等） |
| **cuDNN** | 深度學習原語（卷積、正規化等） |
| **cuFFT** | 快速傅立葉轉換 |
| **cuRAND** | 亂數生成（用於 dropout、權重初始化） |
| **NCCL** | GPU 間集體通訊（分散式訓練） |

### 5.4 CUDA 在 PyTorch 中具體賦能的能力

1. **具 GPU 加速的 Eager Execution** — 在 GPU 上執行 define-by-run 語義
2. **GPU 上的自動微分** — autograd 引擎完全在 GPU 上追蹤運算並計算梯度
3. **混合精度訓練（AMP）** — 透過 Tensor Cores 使用 FP16 搭配 FP32 主權重
4. **大規模分散式訓練** — DDP + NCCL 實現多 GPU / 多節點訓練
5. **編譯器最佳化** — `torch.compile` 從 Python 程式碼生成融合的 CUDA kernel

## 6. 當前關係與未來發展

### 6.1 當前關係

**深度共生：** PyTorch 與 CUDA 已共同演化近十年。NVIDIA 工程師直接嵌入 PyTorch 社群，貢獻包括：

- CUDA Graphs
- AMP（自動混合精度）
- Megatron-LM 整合
- NCCL 分散式訓練後端
- MLPerf 基準最佳化

PyTorch 是 NVIDIA 的**首選深度學習框架**——它是 NVIDIA 用於 MLPerf 提交的框架，也是 NVIDIA AI 軟體堆疊（包含 CUDA、cuDNN、TensorRT 與 Triton Inference Server）的核心組成。

2026 年 7 月宣布的 **7 億次 PyPI 下載**（含 CUDA 支援）強調了這一夥伴關係。

NVIDIA 也是 **PyTorch 基金會**（2022 年在 Linux 基金會下成立）的創始成員，擁有正式的治理影響力。[^wiki-pytorch]

### 6.2 未來方向

1. **Monarch（2025+）** — 叢集級程式設計抽象，讓協調 1000+ GPU 感覺像程式設計單一機器，具備自動分片（sharding）、放置（placement）與容錯（fault tolerance）。
2. **CUDA Graph Trees** — 更高階的圖式執行，組合巢狀圖子區域，進一步降低啟動開銷。
3. **多後端時代** — 雖然 CUDA 仍佔主導地位，PyTorch 現在也將 AMD ROCm 和 Apple MPS 視為一級後端。NVIDIA 的回應是更深入的 CUDA 整合（CUDA 13.x、Tensor Cores、下一代 GPU 架構）以維持效能優勢。
4. **Edge AI 與 ExecuTorch** — 將 PyTorch 帶入行動/嵌入裝置（Arm、Apple Silicon、Qualcomm），CUDA 針對 NVIDIA 邊緣硬體（Jetson 等）進行最佳化。
5. **Agentic AI 與 RL 基礎設施** — TorchForge（強化學習）與 OpenEnv（環境中心）將利用 CUDA 訓練大規模代理系統。
6. **萬億參數模型** — PyTorch 分散式原語（FSDP、tensor parallel、pipeline parallel）與 NVIDIA 硬體進步（H200/B200/下一代 GPU、更快互連）的持續共同演化。[^chintala-design]

### 6.3 競爭格局

CUDA 的專有鎖定（vendor lock-in）正面臨挑戰：

- **AMD ROCm** — 自 PyTorch v1.12（2022 年）起穩定的後端
- **Apple MPS** — Apple Silicon 上的 GPU 加速
- **ZLUDA/HIP** — AMD GPU 的 CUDA 替代方案
- **OpenAI 的 Triton** — kernel 生成層級的競爭

然而，PyTorch 與 CUDA 之間長達十年的共同最佳化——kernel 融合、記憶體分配器調校、集體通訊模式與編譯器專門化——創造了競爭對手仍在努力追趕的持久護城河。

## 7. 結論

PyTorch 與 CUDA 的關係始於 2016 年 PyTorch 的誕生，並在近十年的時間裡共同演化為深度學習領域的事實標準組合。CUDA 為 PyTorch 提供了從底層張量運算到大規模分散式訓練的完整加速基礎，而 PyTorch 則為 CUDA 提供了最重要的應用場景之一。隨著深度學習模型持續增長（從數百億到萬億參數），兩者的共生關係將繼續深化，同時 PyTorch 也逐步擴展到 CUDA 以外的 GPU 後端。

---

[^wiki-pytorch]: Wikipedia. (n.d.). PyTorch. Retrieved 2026-10-09, from https://en.wikipedia.org/wiki/PyTorch

[^chintala-design]: Chintala, S. (2023-12-17). PyTorch's design origins. Retrieved 2026-10-09, from https://soumith.ch/blog/2023-12-17-pytorch-design-origins.md.html

[^nvidia-nccl]: NVIDIA. (n.d.). NCCL (NVIDIA Collective Communications Library). Retrieved 2026-10-09, from https://developer.nvidia.com/nccl

[^pytorch-cuda-graphs-blog]: PyTorch Team. (2021). Accelerating PyTorch with CUDA Graphs. Retrieved 2026-10-09, from https://pytorch.org/blog/accelerating-pytorch-with-cuda-graphs/

[^nvidia-cuda-graphs-docs]: NVIDIA. (n.d.). CUDA Graph Basics — Introduction. Retrieved 2026-10-09, from https://docs.nvidia.com/dl-cuda-graph/latest/cuda-graph-basics/introduction.html

[^pytorch-20]: PyTorch Team. (2023-03-15). PyTorch 2.0 Announcement. Retrieved 2026-10-09, from https://pytorch.org/blog/pytorch-2.0/

[^pytorch_evolution]: PyTorch History Repository. (n.d.). Evolution of PyTorch. Retrieved 2026-10-09, from https://github.com/AlirezaShamsoshoara/PyTorchHistory