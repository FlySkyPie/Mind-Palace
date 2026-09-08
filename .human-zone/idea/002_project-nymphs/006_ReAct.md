# ReAct

```mermaid
graph TD
    A([開始]) --> B{判斷 / 決策}
    B --> C[執行步驟]
    C --> B
    B --> D([結束])

    style A fill:#e1f0ff,stroke:#70a1ff
    style B fill:#e6ffed,stroke:#52c41a
    style C fill:#e6ffed,stroke:#52c41a
    style D fill:#e1f0ff,stroke:#70a1ff
```

ReAct 是現代 (2026) 主流的 LLM 使用模式，也是俗稱的 Agent，然而該模式有幾個致命缺點。

ReAct 的運算複雜度本質上是 $O(N^2)$，因為每一次 LLM Request 的輸入都必須夾帶整個歷史紀錄，越後面的步驟 token 就消耗的越多，而到完成任務為止的總消耗是所有步驟的消耗加總。

因為「必須夾帶整個歷史紀錄」，對於 LLM 上下文窗口的要求變得十分兇猛，必須仰賴更大參數的模型。