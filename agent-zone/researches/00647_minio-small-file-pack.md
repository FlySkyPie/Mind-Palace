# MinIO 對大量小檔案的處理：會不會合併成較大檔案以降低讀寫瓶頸？

## 摘要

MinIO 確實針對「大量小物件」做了最佳化，但它**不會把多個不同的物件合併成同一個檔案**。MinIO 的做法是**把單一小物件的 erasure-coded 分片資料，直接內嵌（inline）到該物件自己的 `xl.meta` metadata 檔案中**，讓「讀一個小物件」只需要開啟一個檔案，而不是過去必須先讀 metadata、再讀資料兩個以上檔案。這個機制降低了每物件所需的 IOPS，是 MinIO 用來對抗小檔案讀寫瓶頸的核心手段[^smallobj][^deepdive]。本文針對這個問題逐層說明。

## 一、問題背景：小檔案為何成為瓶頸

物件儲存系統在處理大量小檔案時的瓶頸，主要來自兩層：

- **metadata 存取成本**：傳統物件儲存每次讀取物件資料前，必須先存取 metadata server（取得映射資訊等），再存取實體儲存。對大檔案而言，每檔一次性的 metadata 延遲幾乎可忽略；但對大量小檔案而言，metadata 存取幾乎會讓資料存取延遲加倍，成為整體系統的瓶頸[^smallobj]。
- **檔案系統層面的 overhead**：MinIO 沒有外部 metadata 資料庫，S3 命名空間本身就對應到 XFS 目錄樹。每個 S3 物件會在 erasure set 的每一顆硬碟上建立一個**每物件子目錄**，內含一個 `xl.meta`（物件過大時另有 `part.1`、`part.2`……等分片檔）[^pepite]。因此一個 16 顆硬碟的 erasure set 中，10 億個物件等於**每顆硬碟各 10 億個目錄**[^pepite]。

MinIO 官方對「小檔案」的定義約為小於 64 KB，並指出越來越多客戶有介於 16 KB 到 1 MB 之間、數量達數十億甚至數兆的檔案[^smallfiles]。這類 workload 的關鍵指標是 IOPS（每秒輸入輸出次數）[^smallfiles]。

## 二、MinIO 的解法：Inline Data（資料內嵌）

### 傳統的磁碟佈局

在舊的、未最佳化的佈局中，每個物件在每顆硬碟上的結構為[^smallobj]：

```
bucket/
  object_path/
    object-name/
      xl.meta        ← metadata
      data-uuid/
        Part.1       ← 物件內容
        Part.2       ← multipart 物件最多可到 10000 個 part
```

這代表**讀取資料至少需要開啟兩個檔案**；**寫入資料至少需要寫兩個檔案加一個資料夾**。對大量小物件而言，這些 overhead 會快速累積[^smallobj]。

### Inline 之後的佈局

MinIO 自 2021 年 4 月 22 日的 release 起加入 inline data 功能，讓小物件把 erasure-coded 的資料分片**直接存在與 metadata 同一個檔案（`xl.meta`）中**[^deepdive][^smallobj]。佈局簡化為：

```
bucket/
  object_path/
    object-name/
      xl.meta        ← metadata + 內嵌的資料分片（不再有 part.N 檔）
```

官方部落格原文指出：「The inline data was added to the format, allowing for small objects to have their data stored in the same file as metadata.」[^deepdive]

### `xl.meta` 內部的結構

`xl.meta` 的內容順序是：metadata 在前、inline data 附加在其後。官方說明：「The inline data is stored after the metadata. This means that in cases where we only need to read the metadata we can simply stop reading the file when we have reached the end of the metadata. This can always be achieved with at most 2 continuous reads.」[^deepdive]

這帶來兩個關鍵好處[^smallobj][^deepdive]：

1. **降低 IOPS**：原本需要開啟物件目錄 + `xl.meta` + `part.1`（至少 2 到 3 個檔案），現在只需讀取 `xl.meta` 一個檔案，讀寫所需 IOPS 約減半。
2. **可無鎖讀取**：一旦內嵌資料被讀入記憶體，即可釋放鎖，後續寫入回應不需與並行寫入互斥。MinIO 原始碼註解即說明，因為 inline 物件的 `xl.meta` 已把資料讀入記憶體，任何後續的 `xl.meta` 變動對整體讀取不造成影響，因此可以 lock-free[^src-erasure]。

### Inline 的門檻值

是否 inline 的判斷依據是**每一顆硬碟上的分片大小（per-drive shard size）**，而不是物件總大小[^src-storageclass]。預設值定義在原始碼中：

- 預設 `inline_block` 為 **128 KiB**（`128 * humanize.KiByte`）[^src-storageclass]。
- 若 bucket 為 versioned，門檻改為 `inlineBlock / 8`，即約 **16 KiB**。原始碼註解說明這是為了「satisfy the same constraints」，避免每次 tag/metadata 異動時重寫大量內嵌資料[^src-storageclass]。

由於判斷的是分片大小，實際能被 inline 的物件總大小取決於 erasure coding 的寬度（data/parity 設定）。官方部落格以較通俗的方式說明：「generally files less than 128KiB are likely to be stored inline with the metadata.」[^smallobj]

MinIO 維護者 Klaus Post 在 GitHub Discussion 中明確表示此門檻**不可設定**（"No. This is not configurable. There are too many factors that play into this for you to make a reasonable decision on it."）[^disc16411]。雖然環境變數 `MINIO_STORAGE_CLASS_INLINE_BLOCK` 存在[^src-storageclass]，但原始碼會在設定值超過 128 KiB 時警告 "performance may degrade for PUT please benchmark the changes"[^src-storageclass]。

### 為何不 inline 全部大小

官方說明，若對所有檔案大小都 inline，會影響物件的 mutation 速度（例如新增 tag 或其他屬性變更時）。因此為了讓更新時間維持合理，並避免重複寫入數 MB 的資料，inline 只套用在小檔案[^smallobj]。原始碼中的 `ShouldInline()` 函式與 `InlineBlock()` 註解都重申了 versioned bucket 為 1/8 門檻的設計[^src-storageclass]。

## 三、MinIO 不會做的事：跨物件打包

必須釐清一個常見誤解：**MinIO 不會把多個不同 S3 物件合併進同一個檔案**。

- 每個物件在 erasure set 的每顆硬碟上，仍然擁有自己獨立的 `xl.meta` 與自己的每物件子目錄[^pepite]。
- 「合併」只發生在**單一物件自身的分片**與其 metadata 之間（一個物件的 shard 被放進自己的 `xl.meta`），並非把不同 key 的物件收集成一個大檔[^smallobj][^pepite]。

Pepitedata 的實地稽核文章指出，這樣做的結果是：1 KiB 物件在 16 顆硬碟、EC 8+8 的 erasure set 上，因為每顆硬碟都要一份 `xl.meta` envelope、XFS 4 KiB 最小區塊對齊、以及每物件子目錄 inode／dirent 的固定開銷，**實際佔用可達約 120 KiB（放大約 8 至 10 倍）**，比某些為小值設計的系統採用 3 份複製還更耗空間[^pepite]。這正是「沒有跨物件打包」的代價。

## 四、針對大量小檔案的其他機制：`.tar` 自動解壓

除了 inline，MinIO 另一個直接針對「大量小檔案」的工具是 **`.tar` 檔自動解壓（auto-extraction）**。

若逐一上傳每個檔案，會因大量的連線建立與拆除、成千上萬次 `PutObject` API 呼叫而產生顯著網路 overhead。常見做法是把所有檔案打包成一個大 tar 檔上傳後再解壓[^smallobj]。

MinIO 支援上傳 `.tar`（可選 Zstandard、lz4、gzip、bzip2 壓縮）並設定 `X-Amz-Meta-Snowball-Auto-Extract=true`，伺服器即會自動解壓，對應的 API 為 `PutObjectExtract`。官方範例[^smallobj]：

```
mc mb play/mybucket
mc cp <path-to-archive>.tar play/mybucket --disable-multipart \
  --attr "X-Amz-Meta-Snowball-Auto-Extract=true"
mc ls play/mybucket
```

官方並提到可從 ZIP archive 下載單一檔案[^smallfiles]。這個機制的重點是：**打包動作發生在上傳端（應用層），以一個大檔批次上傳，而不是 MinIO 在檔案系統底層把多個小物件偷偷合成一個大檔。**

## 五、結論與實務建議

回答原始問題：**MinIO 處理大量小檔案時，會「合併」資料以降低讀寫瓶頸，但合併的層級是「單一物件的資料分片合併進自己的 metadata 檔」，而不是「多個物件合併成一個大檔」。**

具體整理：

| 機制 | 合併的對象 | 目的 | 來源 |
|---|---|---|---|
| Inline data | 單一物件的資料分片 ⊕ 該物件的 `xl.meta` | 讀寫由 2 至 3 個檔案降為 1 個檔案，降低 IOPS | [^smallobj][^deepdive] |
| `.tar` 自動解壓 | 多個小檔案（在上傳端打包） | 減少連線與 API 呼叫 overhead | [^smallobj] |
| 跨物件合併成一大檔 | **無此機制** | — | [^pepite] |

因此，若工作負載是成千上萬甚至數十億的次 KB 到 KB 級物件，只依賴 inline 並不足以解決瓶頸，反而可能因每物件固定 overhead 導致儲存效率反轉[^pepite]。實務上官方與第三方稽核皆建議**在應用層批次化**（tar、Parquet、Avro 等格式）後再上傳，或以其他專為小值設計的儲存引擎處理小記錄[^smallfiles][^pepite]。

值得一提的是，MinIO 官方也指出相較於依賴外部 metadata 資料庫的物件儲存方案，移除該依賴可讓 MinIO 在面對大量小物件時明顯更快，因為 metadata 資料庫在面對海量物件的高並行查詢時容易失去回應[^smallobj]。

## 註腳

[^smallobj]: MinIO. (2025, August 20). *Working with small objects in AI/ML workloads.* Retrieved 2026-10-24, from https://www.min.io/blog/minio-optimizes-small-objects

[^smallfiles]: MinIO. (2025, August 20). *The small files problem: Solutions for big data.* Retrieved 2026-10-24, from https://www.min.io/blog/challenge-big-data-small-files

[^deepdive]: MinIO. (n.d.). *Versioning and metadata deep dive.* Retrieved 2026-10-24, from https://www.min.io/blog/minio-versioning-metadata-deep-dive

[^disc16411]: minio/minio. (n.d.). *Store object content as part of the xl.meta file* [Discussion #16411]. Retrieved 2026-10-24, from https://github.com/minio/minio/discussions/16411

[^src-storageclass]: MinIO. (n.d.). *internal/config/storageclass/storage-class.go* [Source code]. Retrieved 2026-10-24, from https://github.com/minio/minio/blob/master/internal/config/storageclass/storage-class.go

[^src-erasure]: MinIO. (n.d.). *cmd/erasure-object.go* [Source code]. Retrieved 2026-10-24, from https://github.com/minio/minio/blob/master/cmd/erasure-object.go

[^pepite]: Laurenceau, J. (2026, May 17). *Stop using MinIO as a NoSQL database: Why S3 object stores collapse on small-file workloads.* Pepitedata. Retrieved 2026-10-24, from https://pepitedata.com/minio-not-a-nosql-database/

## 來源評析

- **MinIO 官方部落格**（`minio-optimizes-small-objects`、`challenge-big-data-small-files`、`versioning-metadata-deep-dive`）：屬於第一手廠商資料，對機制描述最具權威，但存在行銷傾向，傾向強調優勢、淡化小物件的儲存效率問題。其中 inline 的 128 KiB 數字與原始碼一致，可信度高。
- **MinIO 原始碼**（`storage-class.go`、`erasure-object.go`、`xl-storage-meta-inline.go`）：最可靠的行為證據，可直接驗證 `ShouldInline()` 的門檻邏輯（128 KiB、versioned 為 1/8）與 `SetInlineData()` 的實際呼叫點。本文所引數字皆與原始碼一致。
- **GitHub Discussion #16411**：維護者第一手回覆，確認門檻「不可設定」的官方立場，權威性高，但屬於討論串而非正式文件。
- **Pepitedata 稽核文章**：屬於第三方實地經驗，是唯一明確量化「儲存效率反轉」與 XFS 目錄門檻的來源，提供了官方文件缺失的批判視角；但其結論亦帶有推廣 Apache Cassandra 的商業立場（該站提供付費稽核服務），數字應與原始碼交叉驗證後審慎採用。其文中「inline block 預設 256 KiB」的敘述與 MinIO 現行原始碼的 128 KiB 不符，採用時應以原始碼為準。
