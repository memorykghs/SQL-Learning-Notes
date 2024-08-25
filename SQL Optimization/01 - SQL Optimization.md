# 01 - SQL Optimization
* 文章：https://www.pragimtech.com/blog/sql-optimization/how-is-data-stored-in-sql-database/

本篇專有名詞：
* Data pages
* Rood node
* Leaf nodes
* B-tree
* Clustered index structure

## Outline
How is data physically stored in SQL server
  * Data in SQL Server is stored in a tree like structure
  * Where is the data actually stored

How SQL Server finds a row by ID

## How is data physically stored in SQL server
資料在 table 中是以 row 與 column 的形式被儲存即顯示。實際上在硬體當中，是由一系列的 data pages 來儲存資料的。

data pages：
* SQL Server 中的基本單位
* 一個 data pages 為 8kb
   * 1kb ≅ 1000 bytes，實際上是 1024 bytes
   * 所以 8kb 約為 8000 bytes

![](/images/sql-opt/1-1.png)

#### Data in SQL Server is stored in a tree like structure
table data 在 SQL Server 中以樹狀的結構被儲存。

###### 範例 Employees Table
![](/images/sql-opt/1-2.jpg)

* `EmployeeId` 是 primary key column
* 在預設的情況下會在 `EmployeeId` column 上建立 clustered index
* 代表資料實際儲存在硬體中會依照 `EmployeeId` 進行排序

> 也就是說如果一開始沒有建立 index，等資料表龐大到一定程度才建，資料庫需要一些時間來處理這些資料???

#### Where is the data actually stored
如同上面提到的，資料會被儲存在樹狀結構中的一系列的 data pages 裏面，這種樹狀結構稱為 B-Tree。

> B-Tree = index B-Tree = clustered index

![](/images/sql-opt/1-3.png)

* Tree 的頂端為 Root Node ( 根節點 )
* 最底端稱作 Leaf Node ( 葉節點 )，也是資料實際被儲存的地方
* Root Node 與 Leaf Node 之間的 level 稱為 intermediate levels ( 中間層 )
* intermediate levels 可以有多個，數量取決於這個 table 實際的資料量 ( 行 )

假設 Employees table 有 1200 行資料，且每個 data pages 可以儲存 200 筆資料。

* 在 data pages 中的 rows 會依照 `EmployeeId` 進行排序，因為 `EmployeeId` 是 PK 也是 clustered key。
* 所以在第一個 data pages 裡面儲存了第 1~200 筆資料，第二個儲存 201~400 筆，依此類推

database engine 該如何找到所需的資料?
* root 和 intermediate levels 會記錄 index rows，在這個例子中就是 Employee Id，以及一個指向其他 intermediate level 或 leaf nodes 的 pointer
* 這個樹狀結構中有多個 pointer 幫助 database engine 快速的找到資料的儲存位置

## How SQL Server finds a row by ID
假設我們現在要查詢 `EmployeeId = 1200` 的資料列，會執行 `Select * from Employees where EmployeeId = 1120` 指令。

![](/images/sql-opt/1-4.jpg)

* database engine 會從根節點開始往下依照 index 搜尋，由於 `EmployeeId` 是 1200，所以它會選擇包含 `EmployeeId` 801~1200 的節點進行搜尋
* 如上所述一直往下搜尋直到找到 leaf node，就可以取得想要的資料

依照上圖，在這個例子中只需要三步就可以查詢到指定的資料，所以當有成千上百萬條資料，SQL Server 也可以快速的找到我們要查詢的資料，前提是要有 index。

當我們想要用 Employee name 這種沒有 index 的欄位查詢資料的話，SQL Server 可能就必須要逐條讀許資料庫中的紀錄，效能相對非常的低。

## 小結
* Data pages
  SQL Server 中儲存資料的最小單位，一個 data pages 為 8kb
* Rood node
  根節點，樹狀資料表中最頂端紀錄 index rows 與 pointer 的 node
* Leaf nodes
  葉節點，數狀資料表中最底部的 node，存放實際資料
* B-tree
  SQL Server 預設儲存資料的一種樹狀結構
* Clustered index structure
  指 B-Tree 結構，中間會依照不同的分區指標來將資料分區指標來將資料有
