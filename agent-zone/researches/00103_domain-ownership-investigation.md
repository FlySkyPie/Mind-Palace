# 如何找出網域名稱背後的真實擁有者？

## 概述

當你看到一個網站，想要知道它背後是誰在營運時，有許多方法可以追查。本報告整理了從基本到進階的調查技術，以及相關的法律與倫理注意事項。

## 1. WHOIS 查詢 — 最基本的方法

### 什麼是 WHOIS？

WHOIS（發音為 "who is"）是一種查詢/回應協定（RFC 3912，使用 TCP Port 43），用於查詢網域名稱註冊資料、IP 位址區塊和自治系統的資料庫。它最初於 1970 年代早期由 Elizabeth Feinler 在史丹佛大學的 Network Information Center 為 ARPANET 開發[^whois]。

### 運作方式

- 客戶端連線至 WHOIS 伺服器的 TCP Port 43，發送網域名稱作為文字指令，然後接收文字回應。
- 現代 WHOIS 工具支援 **referral**（轉介）機制：一個「薄型」（thin）註冊局（如 .com 和 .net）只儲存註冊商的 WHOIS 伺服器位址，然後將查詢轉介至註冊商自己的伺服器以取得完整資訊。
- 「厚型」（thick）註冊局（如 .org）則直接儲存所有註冊資料，一台伺服器即可完整回答[^whois]。

### WHOIS 會揭露什麼資料？（未遮蔽時）

- **註冊人**（Registrant）姓名、組織、地址、電話、Email
- **管理/技術/帳務聯絡人**
- **註冊商**名稱與 IANA ID
- **建立、到期、更新日期**
- **DNS 名稱伺服器**與 **DNSSEC** 狀態
- **網域狀態碼**（如 `clientTransferProhibited`、`redemptionPeriod`）
- **註冊商 abuse 聯絡方式**

### 如何執行 WHOIS 查詢

**命令列（Linux/Unix）：**
```bash
whois example.com
```
現代 Linux `whois` 客戶端會根據 TLD 自動選擇正確的 WHOIS 伺服器。

**線上工具：**
- [who.is](https://who.is/) — 簡單的 Web WHOIS、RDAP、DNS 查詢
- [whois.domaintools.com](https://whois.domaintools.com/) — DomainTools WHOIS 查詢
- [whois.icann.org](https://whois.icann.org/) — ICANN 官方 WHOIS 資源
- [search.arin.net/rdap](https://search.arin.net/rdap/) — ARIN 的 RDAP Web 介面（針對 IP/ASN/網域註冊資料）

### 重要變化

**自 2025 年 1 月 28 日起**，gTLD（通用頂級網域）的 WHOIS 已正式淘汰。ICANN 要求註冊局和註冊商改以 **RDAP（Registration Data Access Protocol）** 提供存取。RDAP 回傳結構化 JSON 資料，並支援驗證與隱私功能[^whois]。

## 2. GDPR 與隱私保護 — WHOIS 資料被遮蔽怎麼辦？

### GDPR 的影響（2018 年 5 月）

歐盟的一般資料保護規範（GDPR）與 WHOIS 要求公布個人資料的規定直接衝突。主要後果：
- 多數註冊商現在會**遮隱**（redact）個人聯絡資訊（註冊人姓名、地址、電話、Email），顯示「REDACTED FOR PRIVACY」或代理佔位文字。
- 你能看到的通常只剩：註冊商名稱、註冊商 abuse 聯絡方式、網域狀態、名稱伺服器、日期。擁有者身份被隱藏[^whois]。

### 網域隱私服務

即使在 GDPR 之前，許多註冊商也提供付費的「私人註冊」或「WHOIS 隱私」服務——註冊商自己的聯絡資訊會取代客戶的真實資訊。對執法機關來說，註冊商可以被強制揭露真實擁有者；但對一般大眾而言，追查路徑到此為止。

## 3. 替代方法 — WHOIS 被遮蔽後的追查技巧

### A. 反向 WHOIS（Reverse WHOIS）

與其查詢網域來找擁有者，**反向 WHOIS** 讓你可以用擁有者名稱、Email 或組織來搜尋他們擁有的所有網域。

- **[viewdns.info/reversewhois/](https://viewdns.info/reversewhois/)** — 免費：輸入 Email 或名稱來尋找使用相同資料的其他網域[^viewdns]。
- **DomainTools Reverse WHOIS**（付費） — 專業級的反向 WHOIS，含歷史資料。
- **SecurityTrails**（付費，部分免費查詢可用） — DNS 歷史與反向 WHOIS。

**實用技巧：** 如果你在任何未被遮隱的欄位中找到任何可識別的字串（即使是部分 Email 如 `admin@` 或公司名稱），就用那個字串來搜尋所有相關網域。

### B. DNS 歷史紀錄

當前的 WHOIS/RDAP 資料可能已被遮隱，但**歷史 WHOIS 快照**可能包含隱私功能啟用前或 GDPR 生效前的擁有者資訊。

- **[who.is/history](https://who.is/history)** — 號稱擁有「自 2007 年以來 20 億筆以上的歷史 WHOIS 與 RDAP 快照」，並提供擁有權變更時間軸。
- **[SecurityTrails](https://securitytrails.com/)** — DNS 歷史資料、歷史 WHOIS。
- **DomainTools** — 提供可追溯多年的歷史 WHOIS 記錄。

### C. 憑證透明度日誌（Certificate Transparency Logs — crt.sh）

每個公開信任的 SSL/TLS 憑證都必須記錄在憑證透明度（CT）日誌中。**[crt.sh](https://crt.sh/)** 是一個免費的憑證搜尋引擎。這是一個**非常有用的替代方法**，因為[^ctlogs]：

- 憑證通常會在 subject 或 SAN（Subject Alternative Name）欄位中包含組織名稱、網域名稱，有時還有聯絡資訊。
- 可以按網域、組織名稱或憑證指紋搜尋。
- Chrome 自 2018 年 4 月起要求所有憑證都須有 CT，Firefox 自 v135（2025 年 2 月）起跟進。
- **實用方法：** 前往 `https://crt.sh/?q=example.com` 查看該網域的所有歷史憑證，包括相關的組織名稱和子網域。

其他 CT 工具：[Censys](https://search.censys.io/)、[Cloudflare Radar CT](https://radar.cloudflare.com/certificate-transparency)、[Merklemap](https://www.merklemap.com/)。

### D. Wayback Machine 與網站線索

**[Internet Archive Wayback Machine](https://web.archive.org/)** 儲存了網站的歷史快照[^wayback]。這可以幫助你：

- 查看歷史的「聯絡我們」或「關於我們」頁面，可能包含擁有者資訊。
- 查看先前的網站內容，版權聲明中可能包含公司名稱。
- 在隱私功能啟用之前查看網站。
- **實用方法：** `https://web.archive.org/web/*/example.com` 顯示所有歷史快照。

### E. 社交媒體與網站線索

- 查看網站的「關於」、「聯絡」、「法律聲明」頁面。
- 查看網站頁尾——版權聲明通常包含法人實體名稱。
- 在 **LinkedIn**、**Twitter/X**、**Facebook** 上搜尋該網域——許多企業會將網域連結到社群帳號。
- 使用 **Google dorking** — 例如 `site:example.com "contact"` 或 `inurl:privacy example.com`。
- 查看**網站原始碼**中隱藏的註解、開發者 credit、或分析工具 ID，這些可能連結到特定個人。

### F. Email 發現

如果你不知道誰在營運網站，試試常見的 Email 模式：
- `admin@example.com`、`info@example.com`、`contact@example.com`、`webmaster@example.com`
- 檢查 MX 記錄，了解他們使用哪個 Email 供應商。
- 工具如 [Hunter.io](https://hunter.io/) 或 [Snov.io](https://snov.io/) 可以幫助找出與網域相關的 Email。
- 如果 WHOIS 顯示遮隱的 Email（如 `admin@privacy-protection.registrar.com`），那個位址仍然有效——它會轉發給真正的擁有者。

## 4. 實用工具總整理

### 免費工具

| 工具 | 網址 | 功能 |
|------|------|------|
| **whois CLI** | 內建於 Linux/Unix | 命令列 WHOIS 查詢 |
| **who.is** | https://who.is/ | WHOIS、RDAP、DNS、名稱伺服器查詢 |
| **crt.sh** | https://crt.sh/ | 憑證透明度日誌搜尋 |
| **viewdns.info** | https://viewdns.info/reversewhois/ | 免費反向 WHOIS |
| **ARIN RDAP** | https://search.arin.net/rdap/ | IP/ASN/網域註冊資料 |
| **Wayback Machine** | https://web.archive.org/ | 網站歷史快照 |
| **ICANN WHOIS** | https://whois.icann.org/ | 官方 ICANN WHOIS |
| **DomainTools** | https://whois.domaintools.com/ | WHOIS 查詢 |
| **Censys** | https://search.censys.io/ | 憑證與網路資產搜尋 |
| **Hurricane Electric BGP** | https://bgp.he.net/certs | 憑證搜尋 + BGP 工具包 |

### 付費/商業工具

| 工具 | 專長 |
|------|------|
| **SecurityTrails** | DNS 歷史、反向 WHOIS、IP 資料 |
| **DomainTools** | 歷史 WHOIS、反向 WHOIS、網域情報 |
| **Hunter.io** | 從網域發現 Email |
| **Whoxy** | WHOIS API，含歷史資料 |
| **WhoisXMLAPI** | 反向 WHOIS、網域信譽 |

## 5. 倫理與法律考量

### 法律架構

- **ICANN 要求**（至少在歷史上）準確的 WHOIS 聯絡資料。美國的 **Fraudulent Online Identity Sanctions Act** 規定，如果有人在用於違反商標/著作權法的網域上故意提供虛假 WHOIS 聯絡資訊，即構成違法。
- **GDPR（歐盟）** 超越 ICANN 的要求——WHOIS 中的個人資料必須受到保護。這造成了當前多數 WHOIS 資料被遮隱的局面。
- 自 2025 年 1 月起，gTLD 的 WHOIS 已淘汰，**RDAP 是必要協定**。不過多數 WHOIS 工具仍然彙整 RDAP 資料並以熟悉的格式呈現。

### 調查者的倫理指南

1. **不要將 WHOIS 資料用於垃圾郵件、大量行銷或騷擾**——這就是隱私保護存在的原因，違反此原則可能導致法律責任（尤其在 GDPR 下）。
2. **要有正當目的**——網域擁有權調查在以下情況是正當的：網路安全研究、詐欺調查、智慧財產權執法、新聞調查、或網域購買談判。跟蹤、肉搜或大量 Email 收集則不是正當目的。
3. **尊重速率限制**——WHOIS 伺服器會實施 CAPTCHA 和 IP 速率限制，以防止大量資料抓取。
4. **不要公開肉搜**——如果你發現網域背後的人，除非有強烈的公共利益理由（例如揭露詐騙集團），否則不要公開分享他們的個人資訊。
5. **使用申訴管道**——如果你發現網域涉及詐欺、釣魚或濫用行為，請聯絡註冊商的 abuse 部門（WHOIS/RDAP 中一定看得到），不要自行處理。
6. **考慮當地法律**——GDPR（歐盟）、CCPA（加州）和類似的隱私法可能限制你如何收集、儲存或使用透過這些方法發現的個人資料。

### 調查工作流程

追查網域擁有者的實務步驟：

1. **先查 whois/rdap** — 確認可取得的資訊（至少能看到註冊商、名稱伺服器、日期）。
2. **如果被遮隱 → 查 crt.sh** — 憑證通常會揭露組織名稱。
3. **查 Wayback Machine** — 歷史網站內容可能包含擁有權資訊。
4. **查反向 WHOIS** — 如果你找到任何獨特字串，用它來搜尋。
5. **查 DNS 歷史** — 歷史 WHOIS 快照可能顯示 GDPR 前的擁有者。
6. **查網站本身** — 關於頁面、法律聲明、版權資訊、社群媒體連結。
7. **查 Email 發現** — 嘗試常見模式，或使用 hunter.io。
8. **如果以上都無效，聯絡註冊商的 abuse 部門**，說明你的正當理由。

## 參考資料

[^whois]: Wikipedia. (n.d.). WHOIS. Retrieved 2026-09-20, from https://en.wikipedia.org/wiki/WHOIS
[^ctlogs]: Wikipedia. (n.d.). Certificate Transparency. Retrieved 2026-09-20, from https://en.wikipedia.org/wiki/Certificate_Transparency
[^viewdns]: ViewDNS.info. (n.d.). Reverse WHOIS Lookup. Retrieved 2026-09-20, from https://viewdns.info/reversewhois/
[^wayback]: Internet Archive. (n.d.). Wayback Machine. Retrieved 2026-09-20, from https://web.archive.org/