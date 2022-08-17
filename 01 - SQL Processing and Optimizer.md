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

## Explain Plan

### 分析執行計畫 ( Execution Plans )
* 檢查數據訪問路徑，例如全表掃描、索引全掃描和索引快速全掃描掃描。
* 檢查連接順序和連接類型，例如嵌套循環連接、散列連接和排序合併加入。
* 查看返回的實際行數和估計行數通過查詢。
* 尋找實際和估計之間存在較大差異的計劃步驟行。
* 尋找成本和邏輯讀取顯著不同的計劃步驟。

## 說明計畫簡介
* 執行一段 SQL 語句之前，Oracle 會將這些語法拆成一個一個步驟進行成本分析。
* 每個步驟都會返回相對應的 Row Source ，也就是執行後符合條件的 rows 集合。
* 而這些步驟可以被轉成樹狀圖，一個步驟會是一個 node，根據這些 node 順序可以得知 SQL 執行步驟的先後順序。

###### 參考
* http://blog.itpub.net/14887683/viewspace-1010813/


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
