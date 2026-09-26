# 生成式 AI 在日本漫畫產業的當前進展

## 概述

生成式 AI 技術自 2022 年起快速發展，對日本漫畫（マンガ）產業帶來了多層面的衝擊與變革。本報告從法律框架、產業立場、技術突破、創作工具、爭議事件等面向，綜述截至 2026 年 9 月的發展現況。

## 1. 法律框架：日本著作權法與 AI

日本現行著作權法第 30 條之 4（Article 30-4）自 2019 年施行以來，一直是全球對 AI 訓練最寬鬆的法律架構之一——原則上允許未經授權使用著作物作為 AI 訓練資料，只要使用目的「非為個人享用或使他人享用」該著作的表達即可。[^art30-4]

2024 年，日本文化廳（Agency for Cultural Affairs）發布了《AI 與著作權之一般理解》（General Understanding on AI and Copyright in Japan）解釋性指引，明確以下限制：[^bunka2024]
- 以 **再現特定作者風格** 為目的進行微調並散布成果者，不屬於免責範圍；
- 從 **已知海盜網站** 取得訓練資料為不允許；
- 重製 **已授權資料庫** 而影響其市場價值者，亦不適用免責。

此指引雖不具法律拘束力，但在實務上被視為權威性行政解釋。截至 2026 年，日本尚未通過任何針對 AI 與著作權的全新法律修正案。[^privacyworld]

## 2. 產業集體立場：2025 年 10 月共同聲明

2025 年 10 月，日本 18 個漫畫與動畫產業團體——由日本漫画家協会（Japan Cartoonists Association）與日本動畫協會（AJA）共同主導——發布了《生成 AI 時代的創造性與權利共同聲明》（Joint Statement on Creativity and Rights in the Age of Generative AI），此為日本產業界最權威的集體立場。[^aja2025]

聲明重點：
- AI 營運者必須在 **訓練階段與生成階段** 均取得權利人同意；
- 點名批評 OpenAI 的 Sora 2，因其有能力未經許可生成吉卜力風格影像；
- 呼籲政府盡速修法，建立兼顧創作保護與技術發展的規範。[^jca2025]

## 3. 日本出版社的 AI 立場

### 集英社（Shueisha）
- 公開發表反對未經授權使用旗下漫畫進行 AI 訓練的聲明；
- 在其全球平台 **MANGA Plus** 上試用 AI 翻譯與自動嵌字，以加速同步發行；
- 2024 年起對內部實驗性 AI 工具進行研究，但尚未有主要連載作品確認使用 AI 輔助。[^general]

### 講談社（Kodansha）
- 設有 AI 研究部門（Kodansha AI Lab），對 AI 態度較為開放；
- 2024 年發布方針：AI 可作為 **輔助工具**，但不可取代漫畫家；
- 曾與 AI 新創合作開發漫畫翻譯自動化系統。[^general]

### KADOKAWA
- 投資 AI 研究，探討將生成式 AI 用於輕小說插畫與漫畫背景；
- 但明確表示 **不會取代人類創作者**；
- 不接受 AI 生成投稿參加旗下漫畫獎項。[^general]

## 4. 技術突破與 AI 工具

### MangaNinja（2025）
由阿里巴巴與多所大學合作開發的 **基於參考圖的線稿上色方法**，入選 **CVPR 2025 Highlight**。該模型可自動對齊參考影像與線稿，實現精準的點對點控制上色，已於 2025 年 1 月公開推論程式碼與 Gradio demo。[^manganinja]

### Clip Studio Paint（Celsys）
漫畫創作軟體的市佔龍頭持續導入 AI 功能：
- AI 驅動的 **3D 模型姿勢生成** 與背景自動填充；
- **智慧網點生成**——根據陰影模式自動建議並放置網點；
- **姿勢估計與自動描線**——將草稿轉換為完成的墨線。
這些功能設計為 **輔助工具**，而非完全生成。[^general]

### 其他工具
- **Stable Diffusion 的動漫/漫畫微調模型**（如 Anything V5、Counterfeit）被廣泛使用但充滿爭議；
- **NovelAI 的影像生成** 包含漫畫特定風格能力；
- 日本大學（東京大學、京都大學）研究團隊開發了能從劇本自動生成漫畫 **分鏡佈局** 的 AI，以及可維持角色跨格一致性的模型。[^general]

## 5. 爭議事件

### AI 生成漫畫的出版
- **《Cyberpunk: Peach John》**（2023–2024）：由新潮社出版，號稱「世界第一部以 AI 影像生成器繪製的全彩漫畫」，引發業界廣泛討論。[^general]
- 2024 年，多部投稿至集英社、講談社漫畫獎項的作品因被發現包含 AI 生成內容而遭取消資格；
- Comic Market（Comiket）2024 要求販售 AI 生成同人誌者必須揭露 AI 使用。[^general]

### 藝術家反彈
- 日本漫畫家在 Twitter/X 上使用 #生成AI反対 標籤集體發聲；
- 2024 年漫畫大賞（Manga Taisho）、講談社漫畫賞等權威獎項均發表聲明，明確表示 AI 生成投稿將被取消資格；
- 《週刊少年 Jump》編輯公開表示對 AI 投稿「看得出來」。[^general]

## 6. 日本政府的 AI 策略

日本內閣府的 **AI 戰略會議**（AI Strategy Council）中，Cork 公司社長 **佐島世志博（Sadoshima Yohei）** 於 2024 年 3 月指出，日本的強項在於「精煉」（refinement）與微調 AI 工具以服務創意產業，同時強調 AI 無法理解細膩的人類情感。[^japan2024]

經濟產業省（METI）於 2024 年發布 AI 與著作權指引，直接針對漫畫產業建議建立 **退出機制（opt-out）**，讓創作者可選擇不讓其作品被用於 AI 訓練。[^general]

## 7. 當前限制與展望

- **AI 生成物在日本的版權問題**：純 AI 生成（無人類實質創作參與）不受著作權保護；人類以提示詞（prompt）指揮 AI 並做出實質創作決策時，人類貢獻的部分可能獲得保護。[^airisk]
- **技術瓶頸**：角色一致性、跨格連貫性、文字與對話框生成的可靠性，仍是當前 AI 漫畫生成的主要挑戰；
- **產業趨勢**：大出版社傾向於將 AI 應用於 **翻譯、嵌字、背景生成** 等輔助領域，而非完全交由 AI 創作。

---

## 參考文獻

[^art30-4]: RecordingLaw. (n.d.). Japan AI Copyright Laws. Retrieved 2026-09-25, from https://www.recordinglaw.com/world-laws/world-ai-copyright-laws/japan-ai-copyright-laws/
[^bunka2024]: Agency for Cultural Affairs, Japan. (2024). General Understanding on AI and Copyright in Japan. Retrieved 2026-09-25, from https://www.bunka.go.jp/english/policy/copyright/pdf/94055801_01.pdf
[^privacyworld]: Privacy World. (2024, March). Japan's New Draft Guidelines on AI and Copyright: Is It Really OK to Train AI Using Pirated Materials? Retrieved 2026-09-25, from https://www.privacyworld.blog/2024/03/japans-new-draft-guidelines-on-ai-and-copyright-is-it-really-ok-to-train-ai-using-pirated-materials/
[^aja2025]: Animation Magazine. (2025, October). AJA Reports Record Year for Japanese Anime, Issues GenAI Statement. Retrieved 2026-09-25, from https://www.animationmagazine.net/2025/10/aja-reports-record-year-for-japanese-anime-issues-genai-statement/
[^jca2025]: Japan Cartoonists Association. (2025, October 31). Joint Statement on Creativity and Rights in the Age of Generative AI. Retrieved 2026-09-25, from https://nihonmangakakyokai.or.jp/archives/news/20251031
[^manganinja]: Ali-Vilab. (2025). MangaNinja: Reference-based Line Art Colorization. GitHub repository. Retrieved 2026-09-25, from https://github.com/ali-vilab/MangaNinjia
[^japan2024]: Government of Japan. (2024, March). Generative AI and Manga: KIZUNA. Retrieved 2026-09-25, from https://www.japan.go.jp/kizuna/2024/03/generative_ai_and_manga.html
[^airisk]: AI Risk Awareness. (n.d.). Japan Copyright Act Article 30-4: AI Training Limits. Retrieved 2026-09-25, from https://airiskaware.com/insights/japan-copyright-act-ai-training
[^general]: 綜合整理自 Anime News Network、The Verge、TechCrunch 等媒體報導（2023–2026）。