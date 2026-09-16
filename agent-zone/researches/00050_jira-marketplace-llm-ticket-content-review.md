# Jira Marketplace App 與工具：LLM 自動審查 Ticket 內容品質調查

## 概述

本報告調查 **2025–2026 年間**，市面上能夠使用大型語言模型（LLM）自動審查 Jira Ticket 內容品質的應用程式與工具。與前期調查（00045–00049）聚焦於 FOSS 軟體不同，本次擴大了範圍，涵蓋：

- **Atlassian Marketplace App**（Jira Cloud 外掛）
- **商業 SaaS 產品**
- **n8n 現成工作流程模板**
- **Zapier 整合**
- **開源 CLI 工具**
- **Atlassian 內建 AI（Rovo / Atlassian Intelligence）**

篩選標準為工具需能**自動分析 Jira Ticket 本身的內容品質**（描述完整性、驗收標準清晰度、格式正確性等），而非程式碼審查（Code Review）。

---

## 一、Atlassian Marketplace App（Jira Cloud 外掛）

這是最直接的答案——這些 App 安裝後即在 Jira Cloud 內運作，無須外部基礎設施。

### 1.1 SprintGuard — 語意品質閘門（Semantic Quality Gate）

**定位**：目前最成熟的專用產品，由前 Atlassian Jira 工程師 Baraa Abuzaid 開發。

| 項目 | 內容 |
|------|------|
| **類型** | Jira Cloud App（Marketplace） |
| **核心功能** | Ticket 建立時自動 0–100 分品質評分 |
| **品質檢查** | 模糊描述（字數）、缺少驗收標準、缺少 Story Points、缺少標籤/元件 |
| **自動閘門** | ✅ 分數過低時自動將 ticket 移回 backlog，附帶結構化說明 |
| **其他 Agent** | The Gatekeeper（靜態偵測弱規格）、The Blocker Hunter（即將推出：監控卡住/封鎖的 ticket）、The Architect（即將推出：完整架構審查） |
| **價格** | 30 天免費試用，無須信用卡 |
| **成熟度** | 高—精緻的網站、定價透明、創辦人有 Atlassian 信譽[^sprintguard] |

### 1.2 EthicGuard — AI 驗收標準評分 + Sprint 閘門

**定位**：專注於驗收標準（Acceptance Criteria）品質評分的產品——它**不做生成**，只做評分。

| 項目 | 內容 |
|------|------|
| **類型** | Jira Cloud App（Marketplace） |
| **核心功能** | 自動評分 AC 品質（模糊性、遺漏邊界案例、不可測試的斷言） |
| **自動閘門** | ✅ Definition of Ready：阻止未驗證的使用者故事進入 Sprint |
| **跨 Issue 衝突偵測** | ✅ 捕捉團隊間矛盾的需求 |
| **資料政策** | **零資料保留**—Jira 內容僅在記憶體中分析，從不儲存 |
| **BYO AI Key** | ✅ 支援 Claude、OpenAI、Gemini、自訂端點 |
| **價格** | $9.05/user/月，**10 人以下免費**。透過 Atlassian 帳單支付 |
| **成熟度** | 高—有 ROI 計算機與透明的缺陷成本數學模型[^ethicguard] |

### 1.3 Ticket Quality Auditor — Forge App（零第三方程出境）

**定位**：輕量級智慧層，**完全託管在 Atlassian 內部**，無第三方 AI 資料外洩。

| 項目 | 內容 |
|------|------|
| **類型** | Forge App（Atlassian Runs on Atlassian 合規） |
| **評分機制** | 決定性規則 + 選擇性 AI 深度分析，0–100 分 |
| **審查項目** | 缺少描述、弱 AC、模糊語言、依賴缺口 |
| **AI 建議** | 「Improve Ticket」顯示建議變更，**手動確認後才寫入** Jira |
| **設定** | 管理員可設定專案、Issue Type、自動化規則、評分方式 |
| **版本** | 2.2.0（2026 年 9 月） |
| **成熟度** | 中—較新但隱私定位強，適合資安敏感團隊[^tqa] |

### 1.4 TicketIQ — AI Ticket 評分 + 重複偵測（免費）

**定位**：**免費**的 Marketplace App，使用 Gemini 與 GPT-4o 評估 Definition of Ready。

| 項目 | 內容 |
|------|------|
| **類型** | Jira Cloud App（Marketplace） |
| **評分方式** | AI 驅動的 DoR 評分 |
| **重複偵測** | ✅ 語意重複偵測（pgvector） |
| **儀表板** | 即時 Sprint 健康度分析、多帳號管理 |
| **建議** | AI 提供修復低分 ticket 的建議 |
| **價格** | **免費**（2026 年 5 月發布 v1.0.0） |
| **成熟度** | 早期，但核心功能完整[^ticketiq] |

### 1.5 AI Copilot for Jira（by Celeris/ReqTech）— 需求改寫 + 分析

**定位**：Marketplace 上**最成熟**的 AI Jira 輔助工具（82 次安裝，4.9/5 評分）。

| 項目 | 內容 |
|------|------|
| **核心功能** | 自動審查與改進需求規格、重寫使用者故事、Epics、系統需求 |
| **支援標準** | Agile、SAFe、FDA、INCOSE、ISO、CMMI |
| **審查相關** | **Analyze & Improve** 功能：AI 自動審查評估算求 |
| **其他** | 測試案例生成（Functional、Negative、Gherkin） |
| **價格** | 付費（透過 Atlassian，有免費試用） |
| **版本** | 5.3.0（2026 年 3 月） |
| **成熟度** | 高—此類別中最成熟的 Marketplace App[^aicopilot] |

### 1.6 其他 Marketplace App（部分相關）

| App | 開發者 | 說明 | 相關性 |
|-----|--------|------|--------|
| **AI Acceptance Criteria Generator** | Crosstown Tech | 從描述生成 Given/When/Then AC | ❌ 生成非審查 |
| **Smart AI for Jira** | Infosysta | AI 工作分解、Sprint 規劃、Release Notes | ❌ Sprint 管理 |
| **Copilot for Jira** | PageBrain.ai | GPT-4 寫使用者故事與 AC | ❌ 生成，2.5/5 評分 |
| **Auto-Triage AI** | Sprint Loom | 自動分類/排優先級 | ❌ 分流非審查 |
| **Critto** | — | 評估 AC vs PRs/wiki | ⚠️ 偏向 Code Review |

---

## 二、桌面端工具

### 2.1 Khint — Mac Desktop App（AI 品質 Agent）

**定位**：在 Mac 上本機運作，透過鍵盤快捷鍵（Cmd+Shift+K）執行品質審查。

| 項目 | 內容 |
|------|------|
| **類型** | Mac 桌面 App（非 Marketplace App） |
| **運作方式** | 透過 Jira API 與 MCP 連接，使用儲存的 AI Agent |
| **審查標準** | INVEST、DoR、自訂檢查清單 |
| **結果** | 逐項 pass/fail + 精確缺陷（不可測試的 AC、模糊措辭、缺少理由） |
| **鏈式 Agent** | 可串接第二個 Agent 改寫 ticket 直到通過 |
| **寫回 Jira** | ✅ 以 comment 發佈審查結果 |
| **隱私** | Agent 只看到選取的文字；本地 SQLite 資料庫，永不同步至 Khint 伺服器 |
| **價格** | 免費層：300 credits/月（約 10 次 AI 動作/天） |
| **平台** | 僅 Mac |
| **成熟度** | 中—產品定位清晰，適合個人貢獻者與小型團隊[^khint] |

---

## 三、n8n 現成工作流程

### 3.1 n8n Workflow #16693 — Review Jira ticket quality with OpenRouter GPT

**定位**：**直接命中需求**的現成 n8n 模板，以 OpenRouter（OpenAI 相容模型）審查 Jira ticket 品質。

| 項目 | 內容 |
|------|------|
| **觸發** | Jira Cloud 新 Issue 建立（Story、Bug、Task） |
| **評分標準** | 5 個標準，依 ticket 類型調整（Bug 看重現步驟，Story 看使用者場景） |
| **門檻** | 可設定分數閾值（預設 7/10） |
| **LLM** | 可切換，預設 `openai/gpt-oss-120b:free` |
| **結果寫回** | 分數低於閾值時發佈 coaching comment，高分 ticket **不發 comment**（避免垃圾） |
| **語氣** | 教練式（coaching），非懲罰式 |
| **價格** | **完全免費**—只需 n8n + Jira credentials + OpenRouter key |
| **成熟度** | **可直接上線**的模板，無需從零開發[^n8nworkflow] |

### 3.2 n8n Workflow #8713 — Ticket triage for Jira Service Management with Gemini AI

分類嚴重度、設定元件、發布指引給支援工程師。可延伸至 Zendesk/Freshdesk/ServiceNow[^n8ntriage]。

### 3.3 Loïc Sénéchal 的部落格：以 n8n + AI 自動建立與修正 Jira Ticket

引用現有 Jira ticket → AI 分析 → 重新表述需求 → 建立/更新 ticket → 附上 AI 修正建議 comment。使用 n8n + Gemini CLI + 本地 LLM（LLM Studio）[^loic]。

---

## 四、開源 CLI 工具

### 4.1 Jira Cleanup（vacobuilt/jira-cleanup）

**定位**：目前最成熟的 FOSS 選項，基於政策的 Jira ticket 自動治理工具。

| 項目 | 內容 |
|------|------|
| **類型** | Python CLI 工具（GitHub: vacobuilt/jira-cleanup） |
| **授權** | MIT |
| **Ticket 品質分析** | ✅ 1–10 分評分，提供具體改善建議 |
| **Quiescence 分析** | ✅ 偵測停滯/不活躍的 ticket |
| **LLM 支援** | Anthropic Claude、OpenAI GPT-4、Google Gemini、**本地 Ollama** |
| **模式** | Dry-run 模式，彩色終端輸出 |
| **架構** | 可擴充的 Plugin 架構（可自訂 analyzer） |
| **成熟度** | 中等—1 star，43 commits，活躍開發中（Python 3.11+）[^jiracleanup] |

### 4.2 Jira-automation（Shalini-Mishra31）

以 LangChain + GenAI 生成結構化的摘要、描述與驗收標準。偏向**生成**工具而非審查工具，但可改編使用[^jiraautomation]。

### 4.3 jira-prompts（vivekjain17）

AI Prompt 集合，用於自動生成標準化的 Jira ticket 描述。本質上是 Prompt 庫，非工具[^jiraprompts]。

---

## 五、Zapier 整合

### 5.1 AI by Zapier + Jira Software Cloud

| 項目 | 內容 |
|------|------|
| **觸發** | 可設定 Zap（新 Jira Issue → AI 分析 → 評分 → 條件式更新/comment） |
| **現成模板** | ❌ **無**預先建立的「ticket 品質審查」模板 |
| **完整度** | 需手動搭建 Zap，不如 n8n 的現成模板方便 |
| **適合** | 已使用 Zapier 且只需簡單審查的團隊[^zapier] |

---

## 六、Atlassian 內建 AI（Atlassian Intelligence / Rovo）

| 項目 | 內容 |
|------|------|
| **適用方案** | Jira Cloud Premium（$15.25/agent/月）與 Enterprise |
| **功能** | Virtual Agent（JSM）、Work Breakdown（AI 子任務）、Natural Language → JQL |
| **PR 驗證** | 可驗證 PR 是否滿足儲存的驗收標準 |
| **限制** | **不自動評分 ticket 品質**，不強制 DoR 閘門；非專用品質審查工具 |
| **前提** | 需維護良好的 Confluence 知識庫才能有最佳效果[^rovo] |

---

## 七、綜合比較表

| 工具 | 類型 | 審查方式 | 自動閘門 | 價格 | 成熟度 | 部署方式 |
|------|------|---------|---------|------|-------|---------|
| **SprintGuard** | Marketplace App | 0–100 語意評分 | ✅ 自動移回 backlog | 30 天免費試用 | **高** | Jira Cloud 安裝 |
| **EthicGuard** | Marketplace App | AC 評分 + 缺陷標籤 | ✅ 阻止進入 Sprint | $9.05/user/月，10人以下免費 | **高** | Jira Cloud 安裝 |
| **Ticket Quality Auditor** | Marketplace App（Forge） | 決定性規則 + AI，0–100 | ⚠️ 手動確認 | 付費（Atlassian 管道） | 中 | Jira Cloud 安裝 |
| **TicketIQ** | Marketplace App | DoR 評分 + 重複偵測 | ❌ 儀表板僅顯示 | **免費** | 早期 | Jira Cloud 安裝 |
| **AI Copilot for Jira** | Marketplace App | 需求審查與改寫 | ❌ 手動 | 付費（Atlassian） | **高** | Jira Cloud 安裝 |
| **Khint** | 桌面 App（Mac） | INVEST + DoR Agent | ❌ 手動觸發 | 免費層 300 credits/月 | 中 | 本機 Mac |
| **n8n Workflow #16693** | n8n 模板 | 5 標準評分 + coaching | ⚠️ 條件式 comment | **免費**（自託管） | **可直接上線** | n8n 自部署 |
| **Jira Cleanup** | FOSS CLI | 1–10 品質評分 | ❌ Dry-run/正式 | **免費**（MIT） | 中 | 自部署 |
| **Atlassian Rovo** | 內建 | 工作分解、虛擬 Agent | ❌ 無閘門 | Premium 以上內含 | **非常高** | Jira 內建 |

---

## 八、結論

2025–2026 年間，Jira 生態系中出現了**多個成熟的商業產品**專門解決「LLM 自動審查 Jira Ticket 內容品質」這個需求，這是與前期調查（00045–00049 聚焦 FOSS）最大的差異。關鍵發現：

1. **最成熟的專用 Marketplace App**：**SprintGuard** 與 **EthicGuard**，兩者皆提供自動品質閘門（Quality Gate）功能
2. **零資料外洩方案**：**Ticket Quality Auditor**（Forge App，完全在 Atlassian 內部運作，無第三方 AI 出口）
3. **免費方案**：**TicketIQ**（完全免費）與 **n8n Workflow #16693**（零軟體成本，只需模型 API Key）
4. **開源方案**：**Jira Cleanup** 是最成熟的 FOSS 選項（支援本地 Ollama）
5. **Atlassian 內建**：Rovo 涵蓋部分功能但不專注於品質審查

**若組織非常重視資料安全**：推薦 **Ticket Quality Auditor**（Forge App，零第三方出口）。

**若尋找最成熟的付費方案**：推薦 **SprintGuard**（語意品質閘門）或 **EthicGuard**（專注 AC 品質）。

**若預算有限或偏好自託管**：推薦 **n8n Workflow #16693**（零軟體授權費）或 **Jira Cleanup**（FOSS MIT 授權）。

---

## 參考資料

[^sprintguard]: Baraa Abuzaid. (n.d.). SprintGuard — Semantic Quality Gate for Jira. Retrieved 2026-09-13, from https://www.sprintguard.ai/

[^sprintguard-marketplace]: SprintGuard. (n.d.). Atlassian Marketplace. Retrieved 2026-09-13, from https://marketplace.atlassian.com/apps/3431348326/sprintguard-for-jira

[^ethicguard]: EthicGuard. (n.d.). AI Acceptance Criteria Grading + Sprint Gate. Retrieved 2026-09-13, from https://ethicguard.ai/

[^ethicguard-marketplace]: EthicGuard. (n.d.). Atlassian Marketplace. Retrieved 2026-09-13, from https://marketplace.atlassian.com/apps/914032111/ethicguard

[^tqa]: Ticket Quality Auditor. (n.d.). Atlassian Marketplace. Retrieved 2026-09-13, from https://marketplace.atlassian.com/apps/4140865246/ticket-quality-auditor

[^ticketiq]: TicketIQ. (n.d.). Atlassian Marketplace. Retrieved 2026-09-13, from https://marketplace.atlassian.com/apps/3517075779/ticketiq

[^aicopilot]: Celeris/ReqTech. (n.d.). AI Copilot for Jira. Atlassian Marketplace. Retrieved 2026-09-13, from https://marketplace.atlassian.com/apps/1234191/ai-requirements-copilot-for-jira

[^khint]: Khint. (n.d.). Jira Ticket Quality with AI. Retrieved 2026-09-13, from https://khint.app/use-cases/jira-ticket-quality-ai

[^n8nworkflow]: michael. (n.d.). Review Jira ticket quality with OpenRouter GPT and coaching comments. n8n Workflows. Retrieved 2026-09-13, from https://n8n.io/workflows/16693-review-jira-ticket-quality-with-openrouter-gpt-and-coaching-comments/

[^n8ntriage]: n8n. (n.d.). Ticket triage for Jira Service Management with Gemini AI audit and guidance. n8n Workflows. Retrieved 2026-09-13, from https://n8n.io/workflows/8713-ticket-triage-for-jira-service-management-with-gemini-ai-audit-and-guidance/

[^loic]: Loïc Sénéchal. (2025-10-24). Automating Jira ticket creation and correction with n8n and AI. Retrieved 2026-09-13, from https://loicsenechal.fr/en/blog/2025-10-24-automatisation-developpement/

[^jiracleanup]: vacobuilt. (n.d.). jira-cleanup — Automated Jira ticket governance. GitHub. Retrieved 2026-09-13, from https://github.com/vacobuilt/jira-cleanup

[^jiraautomation]: Shalini-Mishra31. (n.d.). Jira-automation — LangChain + GenAI for Jira. GitHub. Retrieved 2026-09-13, from https://github.com/Shalini-Mishra31/Jira-automation

[^jiraprompts]: vivekjain17. (n.d.). jira-prompts — AI prompts for Jira ticket descriptions. GitHub. Retrieved 2026-09-13, from https://github.com/vivekjain17/jira-prompts

[^zapier]: Zapier. (n.d.). Jira Software Cloud + AI by Zapier Integrations. Retrieved 2026-09-13, from https://zapier.com/apps/jira-software-cloud/integrations/ai

[^rovo]: Atlassian. (n.d.). Jira AI Features (Atlassian Intelligence / Rovo). Retrieved 2026-09-13, from https://www.atlassian.com/software/jira/ai