# ZeroMQ 伺服器端訊息儲存與新用戶端狀態重建能力探討

## 摘要

ZeroMQ 本身是高效能非同步訊息函式庫，**不具備**內建的伺服器端訊息持久儲存或歷史回放（replay）能力。要讓新連線的用戶端重建完整狀態，需在應用層自行實作「快照（Snapshot）+ 增量更新」模式。ZeroMQ 官方指南（zguide）第五章提出的 **Clone Pattern** 即為此類需求的標準解法，透過混合 PUB-SUB 與 ROUTER-DEALER 兩組 Socket，搭配「ICANHAZ?/KTHXBAI」交握協定，讓新用戶端先取得狀態快照，再銜接即時更新串流，達到最終一致性的狀態重建[^clone]。

## 1. ZeroMQ 設計定位：非持久性訊息傳輸

ZeroMQ（ØMQ）是一個輕量級的非同步訊息函式庫，其核心定位是「訊息傳輸層」，而非「訊息代理（Message Broker）」或「持久化佇列」。它不包含內建的：

- 磁碟持久儲存（persistence）
- 訊息歷史紀錄（message history）
- 自動回放（automatic replay）
- 訂閱者狀態管理

Stack Overflow 上已有明確討論指出 ZeroMQ 本身沒有任何持久性機制，端用戶必須自行處理持久化需求[^persistence]。

## 2. 解決方案：Clone Pattern（可靠發佈-訂閱模式）

ZeroMQ 官方指南（zguide）第五章提出了一個完整的參考實作——**Clone Pattern**，專門解決「多用戶端共享最終一致狀態」與「遲到用戶端（late-joining client）完整狀態重建」的問題[^clone]。

### 2.1 架構概覽

Clone Pattern 採用中央伺服器架構，混合四種 Socket 類型：

| Socket 類型 | 用途 | 屬於 |
|---|---|---|
| PUB | 向所有用戶端發佈增量更新 | 伺服器 |
| SUB | 從伺服器接收增量更新 | 用戶端 |
| ROUTER | 處理快照請求（非同步，可尋址） | 伺服器 |
| DEALER | 向伺服器請求完整狀態快照 | 用戶端 |

此外在進階版本（Model Three+）中還引入 PUSH/PULL 讓用戶端也能提交更新，以及 PAIR 用於伺服器內部執行緒通訊。

### 2.2 狀態重建協定：ICANHAZ? / KTHXBAI

新用戶端連線時，透過以下步驟重建狀態：

**步驟一：建立連線**
```c
// 建立 DEALER Socket 用於快照請求
void *snapshot = zsocket_new(ctx, ZMQ_DEALER);
zsocket_connect(snapshot, "tcp://localhost:5556");

// 建立 SUB Socket 用於接收即時更新 (先連線，確保佇列開始累積)
void *subscriber = zsocket_new(ctx, ZMQ_SUB);
zsocket_set_subscribe(subscriber, "");
zsocket_connect(subscriber, "tcp://localhost:5557");
```

關鍵技巧：**先建立 SUB 連線再發送快照請求**。因為 SUB Socket 一旦連線就會開始從 PUB 接收訊息、在核心緩衝區中排隊。當用戶端忙於下載快照時，期間產生的增量更新不會遺失，而是會暫存在 ZeroMQ 的內部佇列中[^clone]。

**步驟二：請求完整快照**
```c
zstr_send(snapshot, "ICANHAZ?");
```

**步驟三：接收所有鍵值對，直到終止訊號**
```c
while (true) {
    kvmsg_t *kvmsg = kvmsg_recv(snapshot);
    if (streq(kvmsg_key(kvmsg), "KTHXBAI")) {
        sequence = kvmsg_sequence(kvmsg);
        break;  // 快照結束
    }
    kvmsg_store(&kvmsg, kvmap);  // 儲存每個鍵值對
}
```

**步驟四：切換至增量更新，並按序號過濾**
```c
while (!zctx_interrupted) {
    kvmsg_t *kvmsg = kvmsg_recv(subscriber);
    if (kvmsg_sequence(kvmsg) > sequence) {
        sequence = kvmsg_sequence(kvmsg);
        kvmsg_store(&kvmsg, kvmap);  // 應用到最新狀態
    } else {
        kvmsg_destroy(&kvmsg);  // 丟棄舊的（已包含在快照中）
    }
}
```

### 2.3 狀態表示與序號機制

- 狀態以 **鍵值對（key-value pair）** 為原子單位，封裝在 `kvmsg` 中。
- 每一筆更新都有全域遞增的 **序號（sequence number）**，用於判斷訊息的時序正確性。
- 快照回傳的 `KTHXBAI` 攜帶當下最新序號，用戶端以此為基準過濾後來增量更新，確保不會重複套用已在快照中處理過的舊訊息。

### 2.4 用戶端也可提交更新（Model Three+）

當用戶端需要修改共享狀態時，透過 **PUSH** Socket 發送更新至伺服器的 **PULL** Socket。伺服器統一指派序號、儲存狀態後，再由 PUB 轉發給**所有**用戶端。這確保了全域順序一致性（global ordering）[^clone]。

## 3. 與 Event Sourcing 的關係

本質上 Clone Pattern 是一種 **Event Sourcing** 風格的狀態同步架構：

- 快照（Snapshot）對應到 Event Sourcing 中的 Snapshot/Materialized View
- 增量更新對應到 Event Log/Event Stream
- 新用戶端重建狀態等同於 Event Sourcing 的 Rehydration/Replay
- 序列號機制保證了事件順序與冪等性

然而 ZeroMQ 本身並未實作 Event Store——真正的持久化儲存（如寫入資料庫、檔案、或專用 Event Store）必須由應用層自行實現。

## 4. 限制與注意事項

| 面向 | 限制 |
|---|---|
| 持久化 | ZeroMQ 本身不持久化資料，伺服器重啟後狀態遺失需由應用層恢復 |
| 快照大小 | 官方設計假設狀態可完全載入記憶體，大量資料需搭配外部資料庫 |
| 最終一致性 | Clone Pattern 保證最終一致性（eventual consistency），非強一致性 |
| Slow Joiner | ZeroMQ SUB 慢加入者問題需靠序號過濾與佇列機制緩解，但無限的歷史深度仍會消耗記憶體 |

## 5. 結論

ZeroMQ **沒有內建的伺服器端訊息儲存與回放能力**。但透過 **Clone Pattern**（混合 PUB-SUB 與 ROUTER-DEALER Socket，使用 ICANHAZ?/KTHXBAI 協定），可以在應用層實現「新用戶端連線時取得完整狀態快照 → 切換至增量更新串流」的架構。這在設計上對應到 Event Sourcing 的 snapshot + event log 概念。若需要真正的持久化儲存，則必須在應用層整合資料庫或專用 Event Store。

---

[^clone]: Pieter Hintjens et al. (2024). Chapter 5 - Advanced Pub-Sub Patterns: Reliable Pub-Sub (Clone Pattern). *ZeroMQ Guide*. Retrieved 2026-09-13, from https://zguide.zeromq.org/docs/chapter5/

[^persistence]: Community (2010). zeromq persistence patterns. *Stack Overflow*. Retrieved 2026-09-13, from https://stackoverflow.com/questions/4059706/zeromq-persistence-patterns

[^latejoiner]: Community (2016). ZeroMQ Clone pattern and late-joining client. *Stack Overflow*. Retrieved 2026-09-13, from https://stackoverflow.com/questions/36135891/zeromq-clone-pattern-and-late-joining-client