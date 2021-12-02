# SQL Processing and Optimizer
## Overview of SQL Processing
SQL 的處理包含了以下幾個主要的組件：
* 解析器（Parser）：檢查語法和語義分析
* 優化器（Optimizer）：依據不同的計算方法，如 CBO 或是 RBO 來確定生成查詢結果最有效的方式
* Row Source Generator 從優化器接收最佳執行計畫並生成 SQL 語句的執行計畫
* SQL 執行引擎（Execution Engine）依執行計畫執行並產生查詢結果

#### Query Optimizer
分為兩種：
* RBO（ Rule-Based Optimizer ）：基於規則的優化器

* CBO（ Cost-Based Optimizer ）：基於成本的優化器（成本：CPU、記憶體、時間……）

## Overview of the Optimizer
Optimizer 可以接收統計資料並分析後，決定使用哪一個最有效率的方法執行 SQL。在 SQL 執行的過程中，這是影響執行效率很重要的一個步驟。

SQL 語句可以通過多種不同的方式執行，包括以下幾種：
* TABLE ACCESS
  * FULL TABLE SCAN 全表掃描
  * TABLE ACCESS BY INDEX ROWID 索引掃描
<br/>

* INDEX SACN：透過 `INDEX` 找到某些 rows 的 `ROWID`，然後讀取 `ROWID` 所指向的 table blocks。
  * INDEX UNIQUE SCAN 索引唯一掃描
  * INDEX UNIQUE SCAN 索引範圍掃描
  * INDEX FULL SCAN 索引全掃描
  * INDEX FAST FULL SCAN 索引快速掃描
  * INDEX SKIP SCAN 索引跳躍掃描
* Nested loops
* Hash joins

## SQL Tune on 執行階段
```
Response time =     CPU time        +       Wait time
                ( 必要的 CPU time +      ( 必要的 Wait time +
                  不必要的 CPU time)       不必要得 Wait time )
``` 

![](/images/2-1.png)

## 參考
* https://docs.oracle.com/cd/B10501_01/server.920/a96533/optimops.htm
* https://ittutorial.org/what-is-the-oracle-optimizer-and-how-cost-based-optimizer-works-oracle-database-performance-tuning-tutorial-3/
* https://docs.oracle.com/cd/B10501_01/server.920/a96533/optimops.htm
