# 在雲端執行自訂 PyTorch 模型 — 解決方案調查

## 概述

開發者若自行訓練或在 GitHub 上找到 PyTorch 模型、並將權重上傳到 HuggingFace，要如何在雲端執行推理（inference）並對外提供 API？本報告調查截至 2026 年 10 月的主要方案，聚焦**個人開發者**可用的低成本選項。

---

## 1. 主要平台比較

### 🏆 Replicate (replicate.com)

透過開源工具 **Cog** 包裝模型：

- 在專案中加入 `cog.yaml` + `predict.py`，定義模型如何載入與推理
- `cog push r8.im/你的帳號/模型名稱` — Cog 自動建立 Docker 映像、產生 OpenAPI schema、佈署到 Replicate
- **定價**：T4 GPU $0.81/hr，A100 $5.04/hr，H100 $5.49/hr；可設定 `min_instances=0` 使閒置成本為 $0
- **優點**：最簡單的推 API 方案，從包裝到上線約 30 分鐘
- **適合**：任何可被 Python function 包裝的模型（影像/音訊/LLM 皆可）[^replicate-custom]

### 🏆 RunPod (runpod.io)

兩種模式：

- **Serverless**：寫 `handler.py`，打包成 Docker，部署為無伺服器端點。每秒計費，無閒置成本。支援 FlashBoot 加速冷啟動、模型快取
- **Pod（租用整台 GPU）**：SSH + Jupyter 存取，最低 RTX A5000 約 $0.27/hr，RTX 3090 $0.50/hr，A100 $1.59/hr
- **優點**：靈活性最高。Serverless 適合 API，Pod 適合開發/互動式除錯/訓練
- **適合**：需要完全控制環境或跑非標準模型的開發者[^runpod-serverless][^runpod-pricing]

### 🏆 Together AI (together.ai)

三層方案：

- **Serverless**：僅限平台上已支援的模型（不開放自訂）
- **Dedicated Endpoints**：保留 GPU 給微調後的模型（需基於支援的基底模型），按 GPU 時數計費
- **Dedicated Container Inference (DCI)**：帶自己的 Docker 容器，完全自訂推理程式碼（需聯絡業務）
- **適合**：微調流行 LLM 且有生產 API 需求的情況；非 LLM 模型不適合[^together-deploy]

### 🏆 Modal (modal.com)

以 Python decorator 定義 GPU function：

```python
import modal
app = modal.App("my-model")

@app.function(gpu="A100")
def predict(input_data: str) -> list:
    import torch
    model = torch.load("weights.pth")
    return model(input_data).tolist()
```

- **定價**：T4 $0.59/hr，A100 $2.50/hr，H100 $3.95/hr；**免費方案**每月 $30 額度
- **優點**：Python-first 開發體驗，無閒置成本，支援訓練與推理
- **適合**：偏好純 Python 無 Docker 的開發者[^modal-pricing]

### 🏆 Hugging Face Inference Endpoints

在 HuggingFace 上選擇模型→選擇 GPU→取得 API 端點。與 HF 生態系深度整合。

- **優點**：若模型已是標準 Transformers/Diffusers 格式，數分鐘即可上線
- **缺點**：非標準模型需要額外包裝[^hf-endpoints]

---

## 2. 方案總覽表

| 平台 | 自訂模型支援 | 閒置成本 | 計費方式 | 最適合 |
|---|---|---|---|---|
| **Replicate** | ✅ Cog | $0（min=0） | 每秒 GPU + 每請求 | 快速上線 API |
| **RunPod Serverless** | ✅ handler.py Docker | $0 | 每秒 GPU | 靈活自訂 API |
| **RunPod Pods** | ✅ SSH 完整控制 | $0.27/hr 起 | 每小時 GPU | 開發/除錯/訓練 |
| **Together AI DCI** | ✅ Docker | 有（保留 GPU） | 每小時 GPU | 自訂推理容器 |
| **Modal** | ✅ Python decorator | $0 | 每秒 GPU+CPU+記憶體 | Python-first 開發 |
| **HF Endpoints** | ✅ HF 標準格式 | 有（保留 GPU） | 每小時 GPU | HF 生態系 |

---

## 3. 如何改編任意 PyTorch GitHub 專案

無論選擇哪個平台，流程大致相同：

### 步驟一：理解模型程式

- 找出模型載入方式（`torch.load`、`from_pretrained`）
- 找出推理函數
- 確認依賴項目

### 步驟二：依平台包裝

**Replicate (Cog)：**
```yaml
# cog.yaml
build:
  gpu: true
  python_version: "3.12"
  python_requirements: requirements.txt
predict: "predict.py:Predictor"
```

```python
# predict.py
from cog import BasePredictor, Path, Input
import torch

class Predictor(BasePredictor):
    def setup(self):
        self.model = torch.load("weights.pth")
        self.model.eval()

    def predict(self, input_data: Path = Input(description="Input")) -> Path:
        result = self.model(preprocess(input_data))
        return postprocess(result)
```

**RunPod Serverless：**
```python
# handler.py
import runpod
import torch
from your_model import load_model

model = load_model()

def handler(event):
    data = event["input"]
    with torch.no_grad():
        output = model(data)
    return {"output": output.tolist()}

runpod.serverless.start({"handler": handler})
```

**Modal：**
```python
import modal
app = modal.App("my-model")

@app.function(gpu="A100", image=modal.Image.debian_slim().pip_install("torch"))
def predict(input_data: str) -> list:
    import torch
    model = torch.load("weights.pth")
    return model(input_data).tolist()
```

### 步驟三：處理模型權重

- **小型模型**：直接打包進 Docker 映像（最簡單）
- **中型模型**：啟動時從 HuggingFace 下載（增加冷啟動時間）
- **大型模型**：使用平台模型快取（RunPod 支援，Replicate 在建構時自動包含）

### 步驟四：處理 HuggingFace 認證

若權重受門檻限制（gated model），透過環境變數設定 `HF_TOKEN` — 所有平台皆支援。

---

## 4. 決策樹

```
你的模型是標準 Transformers/Diffusers 格式？
├── 是 → HF Inference Endpoints（最快）或 Replicate Cog
└── 否 → 你偏好什麼？
    ├── 最簡單上線 → Replicate + Cog（30 分鐘）
    ├── 零閒置成本、完全自訂 → RunPod Serverless
    ├── 純 Python 不想碰 Docker → Modal
    └── 最大控制、最低 GPU 成本 → RunPod Pod（$0.27/hr）
```

---

## 5. 總結建議

對**個人開發者**而言，最務實的三條路徑：

1. **最快上線**：Replicate + Cog。寫 `cog.yaml` + `predict.py`，`cog push` 後即有 API。T4 GPU 成本約 $0.81/hr
2. **最省錢**（開發/除錯）：RunPod Pod，租 RTX A5000 $0.27/hr，SSH 進去安裝任何東西
3. **最佳生產 API**（零閒置成本）：RunPod Serverless 或 Modal。前者 $0 閒置、有 FlashBoot 冷啟動加速；後者每月 $30 免費額度

---

[^replicate-custom]: Replicate. (n.d.). Deploy a custom model. Retrieved 2026-10-01, from https://replicate.com/docs/get-started/deploy-a-custom-model

[^replicate-push]: Replicate. (n.d.). Push a model. Retrieved 2026-10-01, from https://replicate.com/docs/guides/push-a-model

[^cog]: Replicate. (n.d.). Cog: Containers for machine learning. Retrieved 2026-10-01, from https://cog.run

[^runpod-serverless]: RunPod. (n.d.). Serverless quickstart. Retrieved 2026-10-01, from https://docs.runpod.io/serverless/quickstart

[^runpod-overview]: RunPod. (n.d.). Serverless overview. Retrieved 2026-10-01, from https://docs.runpod.io/serverless/overview

[^runpod-pricing]: RunPod. (n.d.). GPU pricing. Retrieved 2026-10-01, from https://www.runpod.io/pricing

[^together-deploy]: Together AI. (n.d.). Choosing a deployment option. Retrieved 2026-10-01, from https://docs.together.ai/learn/choosing-a-deployment-option

[^together-dci]: Together AI. (n.d.). Dedicated Container Inference. Retrieved 2026-10-01, from https://docs.together.ai/docs/dedicated-container-inference

[^modal-pricing]: Modal. (n.d.). Pricing. Retrieved 2026-10-01, from https://modal.com/pricing