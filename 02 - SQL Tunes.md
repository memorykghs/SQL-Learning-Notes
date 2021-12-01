# SQL Tunes
## Overview of SQL Processing
SQL 的處理包含了以下幾個主要的組件：
* 解析器（Parser）：檢查語法和語義分析
* 優化器（Optimizer）：依據不同的計算方法，如 CBO 或是 RBO 來確定生成查詢結果最有效的方式
* Row Source Generator 從優化器接收最佳執行計畫並生成 SQL 語句的執行計畫
* SQL 執行引擎（Execution Engine）依執行計畫執行並產生查詢結果

#### Query Optimizer
分為兩種：
* RBO（ Rule-Based Optimizer ）
  基於規則的優化器
<br/>

* CBO（ Cost-Based Optimizer ）
  基於成本的優化器（成本：CPU、記憶體、時間……）

![](/images/2-1.png)

## 參考
* https://ittutorial.org/what-is-the-oracle-optimizer-and-how-cost-based-optimizer-works-oracle-database-performance-tuning-tutorial-3/
* https://docs.oracle.com/cd/B10501_01/server.920/a96533/optimops.htm
