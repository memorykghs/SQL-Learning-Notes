# 02 - Optimizer 架構與 Access Path

* TABLE ACCESS
  * FULL TABLE SCAN 全表掃描
  * TABLE ACCESS BY INDEX ROWID 索引掃描
<br/>

* INDEX SACN：透過 `INDEX` 找到某些 rows 的 `ROWID`，然後讀取 `ROWID` 所指向的 table blocks。
  * INDEX UNIQUE SCAN 索引唯一掃描
  * INDEX RANGE SCAN 索引範圍掃描
  * INDEX FULL SCAN 索引全掃描
  * INDEX FAST FULL SCAN 索引快速掃描
  * INDEX SKIP SCAN 索引跳躍掃描
  * INDEX JOIN SCAN 
  * Using Bitmap Index
  * Combining Bitmap Index
<br/>

* Nested loops
* Hash joins

## Table Scan
### Full Table Scan
* 在資料庫中沒有下集中指令，資料室無序存放的
* 掃描資料表的所有 rows，但實際上是掃描表中所有的資料塊，所以無論資料塊是否有資料都會掃描。

###### 參考
* https://www.itread01.com/content/1550179297.html

### Table Access By Index RowId
* Oracle 會給資料的每一行附加一個偽列，儲存資料表的名稱、資料位置、一個流水號資訊等等。
* Table Access By Index RowId 通過 rowid 存取整張資料表，所以可以較快取得該行資料。

## Index Scan
* 在索引中，除了儲存每個索引的值外，還儲存相對應的 rowid。
* Idex Scan 可以分成兩步驟，每次都是單獨一次的I/O：
  1. 掃描索引得到 rowed
  2. 通過 rowid 取得具體資料

* 如果資料經限制條件過濾後的資料超過總資料的10%~15%，使用索引效率會下降。索引只適合用在欄位資料唯一值多、且存取資料筆數少的情況
* 根據索引型別與 where 限制條件的不同，可以再細分為以下幾種 Index Scan：
  * Index Unique Scan
  * Index Range Scan
  * Index Full Scan
  * Index Fast Full Scan

### Index Unique Scan
* 存在唯一索引的情況下，存取資料時會使用 Index Unique Scan。
* 搜尋到目標所在的 block 後停止搜索。

### Idex Range Scan
* 在唯一索引上使用運算符 ( 如：>、<、<> 等等 ) 或是對非唯一索引上的資料進行查詢。
* 跟 Index Uniques Scan 不同的地方是會一直搜索直到條件不滿足才停止。

### 

## 參考
* https://www.w3help.cc/a/202104/36679.html
