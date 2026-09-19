# SKILL.md 格式與資料夾結構

## 摘要

SKILL.md 是 Crush AI 助理的技能定義檔，位於技能專屬的資料夾中，由 **YAML frontmatter**（中繼資料）與 **Markdown 主體**（指令）兩部分組成。Crush 實作了開放的 **Agent Skills** 標準（agentskills.io），這套標準已被 Claude Code、Cursor、Codex CLI 等 20 多種 AI 代理工具採用[^crush-repo]。本文說明其格式規範、資料夾結構、載入機制及驗證規則。

## SKILL.md 格式

### Frontmatter

每個 SKILL.md 必須以 YAML frontmatter 開頭，由 `---` 分隔線包圍[^fmt-spec]：

```yaml
---
name: skill-name
description: |
  當使用者提出此描述相符的請求時，應觸發此技能。
---
```

#### 必要欄位[^fmt-spec]：

| 欄位 | 類型 | 說明 | 限制 |
|------|------|------|------|
| `name` | string | 技能唯一識別符 | 最多 64 字元，須符合 `^[a-zA-Z0-9]+(-[a-zA-Z0-9]+)*$`，**須與父目錄名稱相符**[^validation] |
| `description` | string | 技能功能摘要與觸發時機 | 最多 1024 字元，最佳實踐：包含使用者可能說的觸發詞 |

#### Crush 支援的選擇性欄位[^fmt-spec]：

| 欄位 | 類型 | 說明 |
|------|------|------|
| `user-invocable` | boolean | 設為 `true` 時，技能會出現在指令面板 |
| `disable-model-invocation` | boolean | 設為 `true` 時，模型不會自動觸發技能，僅能手動呼叫 |
| `license` | string | 授權識別符 |
| `compatibility` | string | 執行環境需求（最多 500 字元） |
| `metadata` | map[string]string | 任意鍵值對 |

#### Agent Skills 標準定義的選擇性欄位（Crush 自動忽略不支援者）[^fmt-spec][^agensi]：

`version`、`author`、`tags`、`requires`、`when_to_use`、`argument-hint`、`arguments`、`allowed-tools`、`context`（如 `fork`）、`model`、`effort`、`hooks`

### Markdown 主體

Frontmatter 之後即為 Markdown 格式的技能本體，Crush 將其完整注入代理的上下文中作為 XML[^deepwiki]。主體通常包含：

- **# 技能名稱** — 一級標題
- **操作程序** — 步驟化的流程
- **規則與約定** — 硬性閘門、鐵律等強制規範
- **範例** — 可選的輸入輸出展示

## 資料夾結構

### 最小技能[^file-structure]：

```
my-skill/
└── SKILL.md
```

### 完整技能[^anatomy]：

```
my-skill/
├── SKILL.md              # 必要 — 主要指令
├── references/           # 可選 — 按需載入的補充文件
├── scripts/              # 可選 — 可執行程式碼
├── templates/            # 可選 — 檔案範本
└── assets/               # 可選 — 靜態檔案（設定、圖片等）
```

### 多領域組織

當技能支援多個領域／框架時，依變體組織[^creator]：

```
cloud-deploy/
├── SKILL.md              # 共通流程 + 選擇邏輯
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

### 命名慣例[^file-structure]：

- **目錄**：小寫加連字號：`my-skill/`、`code-review/`
- **SKILL.md**：必須大寫，名稱完全相符
- **參考文件**：小寫加連字號：`api-guide.md`
- **腳本**：小寫加副檔名：`generate.py`

## 載入機制：漸進式揭露

Crush 使用漸進式載入（Progressive Disclosure）管理技能內容[^deepwiki]：

| 層級 | 內容 | 載入時機 |
|------|------|----------|
| 1 | `name` + `description`（中繼資料） | 技能發現階段常駐記憶 |
| 2 | SKILL.md 主體 | 技能觸發時載入 |
| 3 | `references/`、`scripts/`、`assets/` | 按需載入 |

- SKILL.md 主體建議保持在 500 行以內；若超過，應增加層次結構並指引下一步[^creator]。

## 技能發現路徑

Crush 掃描以下位置尋找技能（依優先順序）[^crush-repo]：

**全域路徑：**
1. `$CRUSH_SKILLS_DIR`（環境變數）
2. `$XDG_CONFIG_HOME/agents/skills` 或 `~/.config/agents/skills/`
3. `$XDG_CONFIG_HOME/crush/skills` 或 `~/.config/crush/skills/`
4. `~/.agents/skills/`、`~/.claude/skills/`
5. Windows：`%LOCALAPPDATA%\agents\skills\` 等
6. `options.skills_paths` 設定的額外路徑

**專案層級路徑：**
- `.agents/skills`
- `.crush/skills`
- `.claude/skills`
- `.cursor/skills`

### 同名覆蓋規則

當多個技能同名時，**最後發現者獲勝**[^validation] — 使用者技能可覆蓋內建技能。此邏輯由 `internal/skills/skills.go` 的 `Deduplicate()` 函數處理。

## 內建技能

Crush 將技能嵌入二進位檔的 `internal/skills/builtin/` 目錄，編譯時透過 `//go:embed` 包入[^crush-repo]：

```
internal/skills/builtin/
├── crush-config/    # Crush 設定協助
├── crush-hooks/     # Hook 編寫與除錯
└── jq/              # JSON 處理
```

內建技能路徑在 TUI 中顯示為 `crush://skills/<skill-name>/SKILL.md`（非實際網址，由 View 工具原生解析）[^crush-repo]。

## 驗證規則（來自 `internal/skills/skills.go`）[^validation]：

| 檢查項目 | 規則 |
|----------|------|
| `name` 必填 | 不可為空 |
| `name` 長度 | 最多 64 字元 |
| `name` 格式 | `^[a-zA-Z0-9]+(-[a-zA-Z0-9]+)*$` |
| `name` 與目錄相符 | `filepath.Base(skill.Path)` 須等於 `skill.Name`（不區分大小寫） |
| `description` 必填 | 不可為空 |
| `description` 長度 | 最多 1024 字元 |
| `compatibility` 長度 | 最多 500 字元 |
| Frontmatter 格式 | 須以 `---` 開頭並以 `---` 結尾（處理 BOM 與 `\r\n`） |

## 結論

SKILL.md 的格式設計遵循**最小觸發成本、漸進式資訊揭露**原則：中繼資料約 100 字決定是否觸發，主體提供完整程序，附屬資源按需載入。資料夾結構允許彈性擴充腳本與參考文件，透過 `agentskills.io` 開放標準實現跨代理工具的可移植性。

[^crush-repo]: charmbracelet. (n.d.). Crush — GitHub README (Skills section). Retrieved 2026-09-19, from https://github.com/charmbracelet/crush
[^fmt-spec]: Skills Directory. (n.d.). SKILL.md Format Reference. Retrieved 2026-09-19, from https://www.skillsdirectory.com/docs/skill-md-format
[^file-structure]: Skills Directory. (n.d.). Skill File Structure. Retrieved 2026-09-19, from https://www.skillsdirectory.com/docs/skill-file-structure
[^agensi]: Agensi. (n.d.). SKILL.md Format Specification — Complete YAML Frontmatter Reference. Retrieved 2026-09-19, from https://www.agensi.io/learn/skill-md-format-reference
[^deepwiki]: DeepWiki. (n.d.). Project Initialization and Skills — Crush. Retrieved 2026-09-19, from https://deepwiki.com/charmbracelet/crush/7.4-project-initialization-and-skills
[^validation]: charmbracelet. (n.d.). Crush — internal/skills/skills.go. Retrieved 2026-09-19, from https://github.com/charmbracelet/crush/blob/main/internal/skills/skills.go
[^anatomy]: Agensi. (n.d.). SKILL.md Format — Folder Structure. Retrieved 2026-09-19, from https://www.agensi.io/learn/skill-md-format-reference
[^creator]: Crush AI. (n.d.). Skill Creator — Anatomy of a Skill, Progressive Disclosure. Retrieved 2026-09-19, from crush://skills/skill-creator/SKILL.md