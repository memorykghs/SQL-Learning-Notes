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
* 通過 rowid 存取整張資料表，所以可以較快取得該行資料。

## Index Scan
* 什麼是索引?
  * 索引是獨立於表格之外的結構，像是一本書的索引一樣，可以讓資料庫較快存取到目標資料。
  * 索引中除了儲存每個索引的值外，還儲存相對應的 rowid。

* 索引掃描 ( Index Scan ) 可以分成兩步驟，每次都是單獨一次的I/O：
  1. 掃描索引得到 rowid
  2. 通過 rowid 取得具體資料

* 如果資料經限制條件過濾後的資料超過總資料的10%~15%，使用索引效率會下降。索引只適合用在欄位資料唯一值多、且存取資料筆數少的情況
* 根據索引型別與 where 限制條件的不同，可以再細分為以下幾種 Index Scan：
  * Index Unique Scan
  * Index Range Scan
  * Index Full Scan
  * Index Fast Full Scan

### Index Unique Scan
* 存在唯一索引的情況下，存取資料時會使用 Index Unique Scan。
* 資料列的值為唯一值，且搭配 = ( 等號 ) 條件搜尋時會使用此方式搜索資料。
* 搜尋到目標所在的 block 後停止搜索。

### Index Range Scan
* 在唯一索引上使用運算符 ( 如：>、<、<> 等等 ) 或是對非唯一索引上的資料進行查詢。
* 跟 Index Uniques Scan 不同的地方是會一直搜索直到條件不滿足才停止。

### Index Full Scan
* 需要查詢的資料從索引中可以全部得到。
* 與 Index Fast Full Scan 不同的地方是，依照資料索引順序讀。由於資料可能存在於不同的 block 中，所以較耗時。


### Index Fast Full Scan
* 依照 block 順序搜索資料，較少 I/O，速度較 Index Full Scan 快。
* 適用於不需要順序的操作，如 min/avg/sum 等聚合操作。

## 參考
* https://www.w3help.cc/a/202104/36679.html
* https://www.796t.com/article.php?id=214562
* https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/659587/
