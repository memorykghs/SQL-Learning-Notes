# What is SQL Tunes
什麼是不好的 SQL?通常不好的 SQL 代表要用到多的資源，那麼在 SQL 執行過程中沒有效率的 SQL 浪費的資源有哪些?

主要有以下幾項：
1. 過多的解析時間 ( Parse time )
2. 過多的 I/O ( 物理的讀跟寫 )
3. 過多的 CPU 時間
4. 過多的等待時間

## 造成查詢效率低落的原因
* 寫得不好的 SQL ( 廢話 )
* 使用索引 / 未使用索引
* 缺乏索引
* 錯誤的 Join 順序
* 類型錯誤
......
......
* 其他問題等等

## 分析執行計畫 Analyzing Execution Plans
* 檢查數據訪問路徑，例如 `FULL TABLE SCAN`、`INDEX FULL SCAN` 和 `INDEX FAST FULL SCAN`。
* 檢查 `Join Order` 和 `Join Types`，像是 `NESTED LOOPS JOIN`、`HASH JOIN` 以及 `SORT-MAERGE JOIN`。
* 尋找實際和估計之前有較大差異的計畫步驟分析
* 尋找成本含邏輯讀取步驟顯著不同的執行計畫分析