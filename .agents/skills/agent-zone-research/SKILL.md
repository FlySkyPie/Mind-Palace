---
name: agent-zone-research
description: |
  MUST use whenever the user asks to search, research, survey, investigate, look up, explore, or study a topic — even casually ("search for X", "research Y", "survey Z", "find info on...", "look into...", "tell me about... using web search"). This skill enforces the project's agent-zone research rules: file naming convention, lock file protocol, web-search-based fact gathering, APA footnotes with ISO 8601 dates, self-contained reports, zh_TW default language, and critical source reflection. Trigger on ANY research-adjacent phrasing, not only explicit "research" commands.
---

# Agent Zone Research Skill

This skill governs how to conduct research and write reports in this project's `agent-zone/` directory. Follow these rules strictly — they exist so multiple agents can collaborate in the same space without conflicts.

## When this skill triggers

You MUST activate this skill whenever the user's request involves:
- Searching, researching, surveying, investigating, studying, exploring a topic
- "Find information about...", "look into...", "tell me about X (and use web search)"
- "Write a report on...", "document...", "research..."
- Any task where you'd reasonably use web search or fetch tools

## Rules from 000_README.md

These are the core rules you must enforce. Read them fully at `agent-zone/000_README.md` before starting any research.

### 1. File naming

- Output goes to `agent-zone/researches/*.md` only
- Filename format: `(\d){5}_([a-z-]+\.md)` — five-digit number + kebab-case
- The five digits start at `00001` and increment sequentially
- The number is unique within `researches/*.md` — even if files are moved, the number stays claimed

### 2. Lock file protocol (`agent-zone/researches.lock`)

The lock file is a coordination mechanism for multiple agents working simultaneously. Follow these steps precisely:

**Step A — Claim a number:**
1. Read `agent-zone/researches.lock` to find the current largest number
2. Your serial number = largest number + 1
3. Add your number to the lock file (plain number, no prefix) before starting research
4. If the edit fails, another agent claimed it — re-read and retry with the next number

**Step B — Release on completion:**
- If your number is the **only** number in the file (other than `D`-prefixed ones): remove all `D`-prefixed numbers, add `D<your_number>`, and stop
- If other numbers exist alongside yours: change your entry to `D<your_number>` and keep others

### 3. Research methodology

- ALL research MUST be based on web search/fetch tools — never answer from memory or training data alone
- Every fact must be traceable to a cited source
- Critically reflect on sources: consider bias, recency, authority, and corroboration
- NEVER invent sources or citations — if you can't find a source, state that clearly

### 4. Footnotes and citations

Every factual claim must have a footnote using standard markdown footnote syntax:

```markdown
Markdown is a lightweight markup language[^lml].
[^lml]: Lightweight Markup Language (LML) — a notation for formatting text
```

All references must use the **modified APA format** (American Psychological Association) with ISO 8601 dates instead of American date formats:

```
Organization. (n.d.). Article title. Retrieved YYYY-MM-DD, from URL
Author, A. A. (YYYY). Article title. Title of Journal, xx, xx-xx. Retrieved YYYY-MM-DD, from URL
```

See `agent-zone/001_apa-format.md` for detailed APA variants.

### 5. Self-contained reports

Every report is an independent document. DO NOT:
- Reference or assume prior conversation context
- Reference other reports or files
- Assume the reader has any background knowledge

Everything needed to understand the report must be inside the report itself.

### 6. Language and formatting

- Default language: **zh_TW** (Traditional Chinese, Taiwan locale)
- Use Mermaid diagrams for illustrations — never ASCII art
- No ASCII charts, tables, or diagrams

## Workflow summary

1. Read the lock file to determine your serial number
2. Claim the number in the lock file (Step A above)
3. Research using web searches and fetch (critically)
4. Write the report to `agent-zone/researches/<number>_<kebab-name>.md`
5. Release the lock (Step B above)

## Project structure reference

```
agent-zone/
├── 000_README.md         # This file (the rules)
├── 001_apa-format.md     # APA citation format guide
├── researches.lock       # Lock file for serial number coordination
└── researches/           # All reports go here
    ├── 00001_*.md
    ├── 00002_*.md
    └── ...
```