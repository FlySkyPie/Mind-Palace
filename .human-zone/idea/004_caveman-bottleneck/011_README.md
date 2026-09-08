# 穴居人大腦瓶頸

我覺得現在拿 LLM 寫程式的人大部分都搞錯重點了，軟體工程的瓶頸從來不是程式碼本身，而是人員的認知負荷跟溝通成本，不然就不會有人月神話這種東西了，如果軟體開發本身是單純的產生程式碼，那理論上應該可以透過增派人員 scale up，問題是這件事情就是沒辦法，因為溝通成本會成二次方成長，削減增派人員獲得的產能。

"市場到底想要什麼？"
"客戶到底想要什麼？"
"厲害關係人到底想要什麼？"
平常軟體開發九成的時間都在處理這種東西，程式碼只是最後一步。

如果網際網路是關於"如何達成目的的彈藥庫"，LLM 是"達成目的的執行器"，現在的問題是這兩個路徑中間有一個穴居人大腦，作為整個串連系統的瓶頸，所以我想解決的問題是如何增加這個經過穴居人大腦的流量？

## 具體作為

### 翻譯

穴居人大腦使用母語獲取資訊是摩擦與認知負荷最低的方法，於是我最近拿 LLM 來翻譯冷門開源專案的文件：

- https://flyskypie.github.io/lava-docs/
- https://flyskypie.github.io/behaviortrees-docs/
- https://flyskypie.github.io/biomes-docs/
- https://flyskypie.github.io/botcraft-docs/

使用傳統 ETL 工具搭配 LLM 自動且大量翻譯開源專案的 Issue 討論：

- https://github.com/FlySkyPie/github-issue-simple-etl

### Agent 工作流

本專案建立的研究流程，使用 LLM Agent 篩選資訊，總結成認知負荷較低的報告。
