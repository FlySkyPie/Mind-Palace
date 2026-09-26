# 生成式 AI 在日系漫畫的當前進展

> 本報告聚焦於**日系漫畫風格**（黑白線稿、網點、分鏡、連續敘事）的生成式 AI 技術現狀，而非日本政府或產業政策。調查時間：2026 年 9 月。

## 目錄

1. [專用工具生態系](#1-專用工具生態系)
2. [學術研究與開源專案](#2-學術研究與開源專案)
3. [日系漫畫特有挑戰的處理現狀](#3-日系漫畫特有挑戰的處理現狀)
4. [關鍵限制](#4-關鍵限制)
5. [與韓系條漫 AI 的比較](#5-與韓系條漫-ai-的比較)
6. [總結](#6-總結)

---

## 1. 專用工具生態系

### 1.1 專屬漫畫生成平台（商用）

| 工具 | 核心特色 | 收費 | 狀態 |
|---|---|---|---|
| **Comistitch** | 端到端漫畫/條漫管線；角色參考一處定義全頁套用；200+ 向量網點庫（5 級色調階層），支援 mood-aware 自動上網點 | 免費/$5 mo | **營運中** |
| **Anifusion** | 最擅長漫畫/動畫風格輸出；SDXL 核心 + LoRA 訓練 | $9 mo Creator / $24 mo Pro | **營運中** |
| **Comicory** | 故事優先管線（腳本 → 分鏡 → 版面）；20 種藝術風格含漫畫/少年/劇畫；角色一致性在測試中最強 | $4.99 單次 / $9.99 mo | **營運中** |
| **Komiko** | 開源根基；有動畫化管線（線稿上色、補幀、升頻） | 免費（500 zap）/ $9.99 mo Starter | **營運中** |
| **LlamaGen** | 多格生成 +「漫畫工作室」模式；可輸出漫畫轉影片 | 免費 600 額度 + 每日/$18 mo | **營運中** |
| **ReelMind** | 多重影像融合保持角色一致；自定義模型訓練與市集；影片轉漫畫 | Freemium | **營運中** |
| **Clip Studio Paint (Celsys)** | 業界標準漫畫繪圖軟體，2024+ 推出 AI Assistant（線稿自動修正、背景生成、AI 描線輔助）[^celsys] | $5-15/mo | **營運中** |
| **AI Comic Factory** | Hugging Face Space；SDXL 核心；最知名的免費工具 | 完全免費 | **已停維護**（2025.10 repo 封存） |
| **Dashtoon Studio** | 曾為角色一致性最強；有讀者平台生態 | 付費 | **已關閉**（2026.08.15 母公司轉向 AI 影片）[^dashtoon] |

### 1.2 被用於漫畫生成的通用模型

- **FLUX.1 Kontext Pro / Kontext Max / Pro Ultra**（Black Forest Labs）：開源模型領先者；Kontext Pro 擅長敘事一致性；Pro Ultra 支援 4MP 解析度；Kontext Max 可控制對話框字體排版
- **Stable Diffusion XL + ControlNet + IP-Adapter**：DIY 主流方案，搭配 Civitai/Tensor.art LoRA 角色訓練
- **NovelAI Image Generation**：專屬 Anime Diffusion XL 模型，日本使用者多
- **Midjourney v7**：透過 `--cref` 角色參考做封面級漫畫插圖，但需要逐格手動處理

---

## 2. 學術研究與開源專案

### 2.1 重點論文

#### DiffSensei — CVPR 2025

由北京大學（PKU）及 MMLab 團隊提出，為**客製化漫畫生成**框架。核心創新是將多模態大語言模型（MLLM）與擴散模型橋接，使用遮罩交叉注意力（masked cross-attention）實現精確版面控制而不直接傳輸像素。MLLM 配接器可逐格調整角色表情、姿勢與動作。同時發布 **MangaZero** 資料集（43,264 頁漫畫、427,147 個標註分格）。[^diffsensei]

#### MangaDiffusion — 美團 / 復旦大學（2024）

提出**多格漫畫從純文字生成**的架構。關鍵貢獻是「格內與格間資訊交互」（intra-panel and inter-panel information interaction）——一個 transformer block 同時處理單格內部生成與跨格連貫性。支援控制格數、多樣化版面、角色一致性。並建構 **Manga109Story** 資料集（從 Manga109 資料集中提取 109 卷、21,142 頁職業漫畫家作品）。[^mangadiffusion]

#### 其他論文

- **"Towards Intelligent Manga Generation"**（IEEE, 2025）：綜述 GAN、CNN、擴散模型、MLLM 在漫畫自動化中的應用。
- **"Anime Generation through Diffusion and Language Models: A Comprehensive Survey"**（2025）：動畫/漫畫生成的最新全面調查。
- **"AI-driven Background Generation for Manga Illustrations"**（ORESTA）：以深度學習進行漫畫背景生成與風格遷移。
- **"Collaborative Comic Generation: Integrating Visual Narrative Theories"**（CREAI 2024）：人機協作漫畫生成，允許在生成過程中修改細節。

### 2.2 開源專案

| 專案 | 說明 | 授權 |
|---|---|---|
| **MangaGen** | 端到端管線：故事提示 → 多 LLM 故事規劃 → 角色 DNA 系統 → 動態版面引擎（15+ 模板）→ 圖片生成 → 頁面合成 → 人臉感知對話框 → PDF 匯出[^mangagen] | MIT |
| **ComicMaster** | ComfyUI 為基礎的漫畫生成管線：支援對話框、頁面版面、匯出[^comicmaster] | — |
| **Manga109 資料集** | 東京大學 Aizawa-Yamasaki-Matsui 研究室發布的最大開源漫畫資料集，109 卷專業作品[^manga109] | 學術使用 |

---

## 3. 日系漫畫特有挑戰的處理現狀

### 3.1 網點處理 — ✅ 大致已解決

**Comistitch** 擁有最成熟的 AI 網點系統：[^screentones]

- **5 級色調階層**：純白 → 淡調（10-20%）→ 中調（~40%）→ 深調（60-70%）→ 純黑
- **mood-aware 映射**：AI 讀取情緒標籤（動作/浪漫/恐怖/憂鬱）自動套用對應網點風格：
  - 動作 → 速度線網點、硬邊圓點
  - 浪漫 → 柔化漸變網點、閃光疊層
  - 恐怖 → 雜訊紋理、不規則斜線
  - 憂鬱 → 輕斜線、淡出漸層
- **200+ 向量網點庫**：點密度 10-70%、漸層、斜線、交叉斜線、雜訊、速度線、裝飾紋樣。向量格式不因印刷 DPI 產生鋸齒
- **處理速度**：6 格版面自動上網點約 30-60 秒（傳統手工作業 1-2 小時）

**現有不足**：每頁仍有 1-2 格需要人工修正。傳統手繪的「觸感」與客製質感 AI 仍無法取代。

### 3.2 分鏡與版面 — 🟡 有進展但仍受限

- **MangaDiffusion** 明確處理格內與格間資訊交互，可控制格數與版面多樣性
- **MangaGen** 提供 15+ 版面模板由 LLM 動態選取（動作→斜角分割、對話→頭像並排、轉折→跨頁）
- **Comistitch** 可從腳本自動配置版面並逐頁選擇版型

**關鍵限制**：AI **無法理解敘事流與視覺引導**。模板化版面可行，但角色跳出分格、自訂分格形狀、格線寬度變化的敘事意圖——這些仍然是必須人類主導的工作。

### 3.3 描線與墨線 — 🟡 部分解決

- **Clip Studio Paint AI Assistant**：提供 AI 線稿自動修正與 AI 背景生成
- **Komiko**：有線稿上色管線
- **主流做法**：多數工具以訓練在漫畫資料上的擴散模型產生「模擬描線」效果，而非真正的向量線稿

**限制**：真正可控制線條粗細、筆觸動態、刻意風格化的描線尚無法由 AI 達成。

### 3.4 角色一致性跨頁 — 🔴 最困難的問題

**根本原因**：擴散模型沒有「頁與頁之間的記憶」。每次生成是統計獨立的。文字描述是視覺資訊的有損編碼器。

**四層一致性方案**（由弱到強）：[^consistency]

1. **提示詞一致**：每格貼上相同角色表（免費但最弱）
2. **參考圖一致**：使用 Midjourney `--cref`、ControlNet、IP-Adapter
3. **管線級角色參考**：角色在系統中定義一次，自動注入每格（Comistitch、Comicory）
4. **風格鎖 + 角色鎖**：專案級不可變參考

**2026 年現狀**：
- 管線級工具可達 **~20+ 格/章**維持可接受的一致性
- LoRA 訓練 15-30 張圖片有效但「耗時極高」——「等你訓練完，這個角色可能已經退場了」
- IP-Adapter 可單張參考但效果取決於條件
- Adobe Firefly Custom Models（beta）可訓練 10-30 張圖片自定義模型

**真實基準**（來自 Kuro.boo 的關鍵分析）：「大約 80% 成功率，也就是每 5 格就有 1 格明顯失敗。訓練 LoRA 花費的時間足夠手繪好幾章漫畫了。」[^kuroboo]

> **Comistitch 的關鍵洞察**：「超過 80% 的回報一致性問題，根本原因都是該格的提示詞中角色描述被縮寫或改寫，而非完整原樣貼上。」

---

## 4. 關鍵限制

### 4.1 尚無法突破的鴻溝

1. **情感與脈絡細微差別**：AI 無法自主畫出「假裝驚訝但隱藏悲傷」的表情——這正是漫畫的核心表現力 [^kuroboo]
2. **選擇性變形與風格化**：AI 無法刻意將角色變形來製造衝擊性名場面（如《ONE PIECE》《JoJo 的奇妙冒險》式的誇張表情）
3. **敘事流理解**：AI 無法理解視覺引導、節奏控制、翻頁驚喜——模板版面可行但有意義的格形變化不可行
4. **角色一致性僅 80%**：最佳工具跨章也只能達 ~80%，剩下 20% 需大量手動修正
5. **勞動型態轉移而非消失**：AI 漫畫生產將**繪圖的體力勞動**轉換為**引導 AI 的認知負荷**——角色從工匠變成導演，而導演 AI 本身就是一項困難的新技能 [^kuroboo]
6. **版權不確定性**：純 AI 生成作品在美國無法進行著作權登記（2023 年判例），需要「實質人工作品」（30%+ 修改）才能獲得保護
7. **平台限制**：Amazon KDP 要求揭露 AI 生成；Pixiv 需 AI 標籤；Webtoon Canvas 部分地區禁止 AI
8. **訓練數據訴訟**：Sarah Andersen 集體訴訟、Getty Images 17 億美元訴訟持續進行中
9. **「AI 垃圾」飽和**：低品質 AI 漫畫湧入平台造成讀者疲勞與類別貶值
10. **業界反彈**：WIT Studio、東映等主要動畫工作室在 2026 年已公開道歉或收回 AI 使用

### 4.2 2026 年的誠實評估

> 「2026 年的今天，雖然圖片生成 AI 在單張插畫已達專業水準，但漫畫——作為一種連續敘事媒介——的門檻仍然很高。從『把 AI 當作便利工具』到『真正自動化』之間，存在一道深淵。目前的 AI 漫畫生產僅是將繪圖的體力勞動置換成了引導與控制 AI 的認知勞動。」—— Kuro.boo, 2026 [^kuroboo]

---

## 5. 與韓系條漫 AI 的比較

### 5.1 維度對照

| 維度 | 日系漫畫 | 韓系條漫/Webtoon |
|---|---|---|
| **色彩** | 黑白 + 網點 | 全彩（標準） |
| **閱讀方向** | 右至左（跨頁） | 左至右（垂直捲動） |
| **頁面結構** | 方格網格 | 無限垂直長條、長留白 |
| **AI 挑戰** | 網點密度、精確 B&W 線條、角色比例漂移更明顯 | 跨集色彩一致、服裝一致 |
| **AI 優勢** | B&W 可隱藏部分瑕疵；渲染需求簡單 | 彩色掩蓋細微漂移；垂直版式對格線尺寸較不敏感 |
| **主流工具** | Anifusion、Clip Studio Paint AI、MangaDiffusion | Dashtoon（已關閉）、Webtoon AI Painter、Comistitch |
| **學術研究** | MangaDiffusion、DiffSensei（專用資料集） | 較少學術論文，多為平台原生工具 |

### 5.2 一致性挑戰差異

Comistitch 指出：「漫畫的高對比黑白風格讓面部比例漂移**更明顯**；條漫的垂直捲動格式與彩色飽和度可掩蓋細微漂移，但跨集服裝不連貫問題**更嚴重**。」[^comparison]

### 5.3 AI 成熟度比較

**韓系條漫 AI 較成熟**的原因：
- 全彩對擴散模型更容易（訓練資料更多，不需專門技術）
- 垂直捲動格式較寬容（不需精確版面構成）
- Naver（LINE Webtoon）大量投資官方 AI 工具（Webtoon AI Painter）
- 韓國條漫工作室已積極採用 AI 於生產管線

**日系漫畫 AI 相對落後**的原因：
- 網點生成需要專門技術（非一般圖片生成）
- 右至左分格流對 AI 更難學習
- 黑白美學需要更高精度
- 傳統漫畫文化對 AI 接受度較保守

---

## 6. 總結

生成式 AI 在日系漫畫領域的進展在 2026 年呈現以下態勢：

- ✅ **可勝任**：背景生成（傳統最耗時）、速度線/效果線、重複質感、快速原型與分鏡草稿、角色渲染（管線設定正確時）
- ❌ **仍無法**：情感細膩度與潛文本、刻意變形與風格化、敘事流與視覺節奏、真正角色一致性（僅 ~80%）、原創性與創作主體性

2026 年最大的趨勢是：**AI 漫畫工具市場正被 AI 影片吞噬**。過去兩年已有三個主要工具關閉或轉向 AI 影片產品，因為投資人認為漫畫市場不如影片市場大。存活下來的工具要嘛往動畫管線多元化（Komiko、ReelMind），要嘛深化角色一致性和網點能力（Comistitch、Anifusion）。

從創作本質來看，AI 尚未改變漫畫的核心門檻——**連續敘事的意圖與控制**。它改變了勞動形式（畫圖 → 指令），但沒有消除勞動量，只是轉移了勞動類型。

---

## 參考資料

[^diffsensei]: Jianzun Wu, Chao Tang, Jingbo Wang, et al. (2025). DiffSensei: Bridging Multi-Modal LLMs and Diffusion Models for Customized Manga Generation. *CVPR 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2412.07589

[^mangadiffusion]: Siyu Chen, Dengjie Li, Zenghao Bao, et al. (2024). MangaDiffusion: Multi-panel Manga Generation with Layout-controllable Diffusion. Retrieved 2026-09-25, from https://arxiv.org/abs/2412.19303

[^mangagen]: Barun-2005. (n.d.). MangaGen: End-to-end manga generation pipeline. Retrieved 2026-09-25, from https://github.com/Barun-2005/manga-gen-ai-pipeline

[^comicmaster]: McMuff86. (n.d.). ComicMaster: AI comic generation pipeline. Retrieved 2026-09-25, from https://github.com/McMuff86/ComicMaster

[^manga109]: Aizawa-Yamasaki-Matsui Lab, The University of Tokyo. (n.d.). Manga109 Dataset. Retrieved 2026-09-25, from https://manga109.github.io/manga109-project-website/en/index.html

[^celsys]: Anime News Network. (2024). Over 60% of professional mangaka now use AI tools for at least one stage of production. Retrieved 2026-09-25, from source cited in Comistitch character consistency guide.

[^screentones]: Comistitch. (n.d.). AI Manga Shading and Screentones Guide. Retrieved 2026-09-25, from https://comistitch.com/blog/manga-shading-and-screentones-with-ai/

[^consistency]: Comistitch. (n.d.). Character Consistency in AI Comics: The Ultimate Guide. Retrieved 2026-09-25, from https://comistitch.com/blog/character-consistency-ai-comic-ultimate-guide/

[^comparison]: Comistitch. (n.d.). Manhwa vs Manga vs Webtoon Comparison. Retrieved 2026-09-25, from https://comistitch.com/blog/manhwa-vs-manga-vs-webtoon/

[^kuroboo]: Kuro.boo. (2026). The Reality of AI Manga Production: A 2026 Analysis. Retrieved 2026-09-25, from https://kuro.boo/blog/reality-of-ai-manga-production-2026-analysis/?lang=en

[^dashtoon]: Comicory. (2026, September). 10 Best AI Comic Generators 2026. Retrieved 2026-09-25, from https://www.comicory.com/blog/best-ai-comic-generators-2026

[^llamagen]: LlamaGen.ai. (n.d.). Best AI Manga Generator 2025. Retrieved 2026-09-25, from https://llamagen.ai/blogs/best-ai-manga-generator-2025-create-publish-manga-comics

[^reelmind]: ReelMind.ai. (n.d.). AI Manga: Next-Gen Manga Creation Tools. Retrieved 2026-09-25, from https://reelmind.ai/blog/ai-man-hua-next-gen-manga-creation-tools