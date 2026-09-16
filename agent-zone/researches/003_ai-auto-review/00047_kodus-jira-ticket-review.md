# Kodus AI 是否具備自動審查 Jira Ticket 內容的能力？

## 調查結論

**否，Kodus AI 不具備自動審查 Jira Ticket 本身內容（如品質、完整性、清晰度）的能力。**

Kodus 的 Jira 整合是**單向**的——它**從 Jira 拉取 Ticket 內容**（標題、描述、驗收條件）來輔助 PR 審查，而非反向對 Ticket 內容進行分析或提出改善建議[^readme][^jira-doc]。

---

## 一、Kodus AI 簡介

Kodus 是一款開源 AI Code Review 工具，採用混合 AST + LLM 架構以減少純 LLM 方案的誤判率。其核心特點包括[^readme][^docs]：

- **模型無關**：支援 Claude、GPT-5、Gemini、Llama、GLM 等模型，使用者自備 Key，無平台加價
- **可自託管**：支援 Docker Compose、Helm 部署，亦有雲端方案
- **支援 4 大 Git 平台**：GitHub、GitLab、Bitbucket、Azure DevOps
- **整合專案管理工具**：Jira、Linear、Notion
- **商務邏輯驗證**：比對 PR diff 與 Jira/Linear/Notion Ticket 的需求

---

## 二、Jira 整合實際行為

Kodus 的 Jira 整合經由 Plugin 系統（Settings → Plugins → Jira → Connect）設定，支援 OAuth 或 API Token 連接 Jira Cloud 與 Jira Data Center[^jira-doc][^plugins]。

整合提供三個功能：

### A. 自動商務邏輯驗證（Business Logic Validation）

當 PR 關聯至 Jira Ticket 時，Kodus 自動[^biz-logic]：

1. 抓取 Ticket 標題、描述與驗收條件
2. 比對 PR diff 與需求
3. 回報缺少的實作、範圍不符與落差，附帶嚴重等級（`MUST_FIX`、`SUGGESTION`、`INFO`）

### B. 隨需驗證

在 PR 留言中可透過指令觸發驗證[^biz-logic]：

```
@kody -v business-logic https://your-org.atlassian.net/browse/PROJ-123
```

### C. MCP 整合規則

Kody Rules 中可使用 MCP functions 來抓取任務上下文、檢查 Issue 狀態、交叉比對需求[^plugins]。

---

## 三、Kodus 對 Ticket 的態度：消費而非審查

Kodus 將 Ticket 分類為四種品質等級（Complete / Partial / Minimal / Empty），但這個分類**僅用於內部決定驗證深度**，並不會對開發者或 PM 提出 Ticket 改善建議。也就是說[^biz-logic]：

```
Jira Ticket (title, description, acceptance criteria)
    → 流入 → PR Code Review（比對 diff 是否吻合）
    ↛ 不會 → 分析 Ticket 品質或建議改善
```

此外，Kodus 有一個已知的 Bug（Issue #1725）[^issue-1725]：當 Jira fetch 回傳空資料時，Kody 曾在 PR comment 中**幻覺出不存在的 Ticket 內容**。此問題被視為 Bug 而非功能，並在 PR #1749 中修復——這進一步證明審查 Ticket 內容並非 Kodus 的設計目標。

---

## 四、Kodus 的 Issue/Ticket 管理能力

Kodus 內部有一個「Kody Issues」功能，但它並非 Jira Ticket 管理系統[^kody-issues]：

| 功能 | 說明 |
|---|---|
| 自動追蹤 | 已關閉 PR 中未實作的建議自動轉為 Issue |
| 自動關閉 | 未來 PR 實作該建議後自動標為已解決 |
| 篩選分類 | 依狀態、嚴重度、類別、儲存庫過濾 |

Kody Issues **不會**建立、更新或管理 Jira Ticket。

---

## 五、總結

| 面向 | 結果 |
|---|---|
| Kodus 是否能審查 Jira Ticket 的內容品質？ | ❌ 否 |
| Kodus 能否拉取 Jira Ticket 內容來輔助 PR 審查？ | ✅ 是 |
| Kodus 能否自動對 Ticket 內容提出改善建議？ | ❌ 否 |
| Kodus 能否建立/更新 Jira Ticket？ | ❌ 否 |

若需求是「自動審查 Jira Ticket 內容的品質與完整性」，Kodus 無法滿足此需求。目前市場上也沒有廣泛採用的工具專門針對 Jira Ticket 內容進行自動品質審查——這仍屬於人工（如 PR peer review、PO 驗收）的領域。

---

[^readme]: Kodus Tech. (n.d.). Kodus AI — Open-source AI Code Review Agent. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

[^docs]: Kodus Tech. (n.d.). Kodus Documentation Overview. Retrieved 2026-09-16, from https://docs.kodus.io/

[^jira-doc]: Kodus Tech. (n.d.). How to Connect Jira to Code Review. Retrieved 2026-09-16, from https://docs.kodus.io/knowledge_base/en/how-to-connect-jira-to-code-review

[^biz-logic]: Kodus Tech. (n.d.). Business Logic Validation. Retrieved 2026-09-16, from https://docs.kodus.io/how_to_use/en/code_review/business_logic_validation

[^plugins]: Kodus Tech. (n.d.). Plugins (MCP) Documentation. Retrieved 2026-09-16, from https://docs.kodus.io/how_to_use/en/code_review/plugins

[^kody-issues]: Kodus Tech. (n.d.). Kody Issues Overview. Retrieved 2026-09-16, from https://docs.kodus.io/how_to_use/en/issues/overview

[^issue-1725]: Kodus Tech. (n.d.). Issue #1725 — Kody hallucinated missing ticket content in PR comment. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai/issues/1725