# 03 - What is heap table and full table scan
* 文章：https://www.pragimtech.com/blog/sql-optimization/what-is-heap-table/

在這個章節中將會提到：
1. 什麼是 heap table? 如何建立以及如何使用?
2. 在此過程中，順帶理解 Table Scan 概念
3. 從範例看使用 full table scan 比 table index 好
4. 強制讓 database server 使用特定的索引來執行查詢

以上這些概念對於 SQL 調教來說非常重要。

## Outline
What is heap table
  * What if we don't have a clustered index on a table?
  * Heap table without a clustered index is called heap

SQL Script to create Gender table
Non-clustered index on heap table
  * Force a query to use an index in sql server

How ara data rows stored in the heap
  * How to create and remove heaps

## What is heap table
我們將透過 `Employees` 表作為範例來理解。`EmployeeId` 一樣是 PK，在預設情況下，PK 上面會建立 clustered index，代表資料是依照 `EmployeeId` 排序被儲存的。

![](/images/sql-opt/3-1.jpg)

#### What if we don't have a clustered index on a table?
clustered index 決定資料列實際上該以什麼順序被儲存在記憶體中，但如果沒有 clustered index，資料列就無法保證以什麼特定的順序被儲存。

這種類型的 table 被稱作 heap ( 堆 )。

![](/images/sql-opt/3-2.png)

#### Heap table without a clustered index is called heap
heap 代表的是一個沒有 clustered index 的 table，但同時他可能有很多個 non-clustered indexs。不過在這個例子中，我們假設 heap 中沒有任何的 non-clustered index。

如果在 heap 中沒有 non-clustered indexes，那麼將會使用 Table Scan 查詢資料。Table Scan 代表 database server 需要掃描每一個在 table 中的 row 來找到我們要的資料。

## SQL Script to create Gender table
建立一張沒有 clustered index 與 non-clustered indexes 的 table `Gender`。

```sql
Create Table Gender
(
	GenderId int,
	GenderName nvarchar(20)
)
Go

Insert into Gender values(1, 'Male')
Insert into Gender values(3, 'Not Specified')
Insert into Gender values(2, 'Female')
```

使用系統儲存的 `sp_helpindex` stored procedure 檢查這張 table 是否有任何的 index。
```sql
sp_helpindex Gender
```

接著使用 `Include Actual Execution Plan` 執行下列查詢 SQL。
```sql
Select * from Gender where GenderName = 'Male'
```

從 execution plan 可以看出它使用 table scan 來尋找資料。
![](/images/sql-opt/3-3.png)

根據下面的狀態：

* `Number of rows read` = 3，代表 SQL server 讀取過的資料數量
* `Actual number of rows of all executions` = 1，則是代表實際上我們要的資料只有一筆
* 結論就是，為了取得我們要的這一筆資料，SQL server 需要讀取三筆資料

![](/images/sql-opt/3-4.png)

不過 `Estimated Subtree Cost` 是 0.0032853，看起來不差。

所以要記住以下 3 個關鍵點：
1. 如果查詢時使用了 table scan，代表沒有任何 indexes 可以使用，讓從 heap table 查詢資料變得更有效率
2. 從效能的角度來看，table scan 的效能並非都是差的或不好的，特別是像範例中這種資料少的 table
3. 在 `Gender` 這種資料非常少的 table 當中，SQL Server 可能會使用 table scan 而不是 index 進行查詢，因為 table scan 效能可能比 index 好

## Non-clustered index on heap table
在 `Gender` table 上建立一個 non-clustered index。

```sql
Create nonclustered index IX_Gender_GenderName
on Gender(GenderName)
```

然後執行查詢並查看 execution plan。
```sql
Select * from Gender where GenderName = 'Male'
```

![](/images/sql-opt/3-5.png)

所以即使我們在 GenderName 欄位上加上 non-clustered index，SQL server 仍然使用 Table Scan 查詢資料。因為 database engine 知道 Table Scan 的效能會比使用 index 來的好，特別是在資料量非常少的時候。

#### Force a query to use and index in sql server
SQL Server query optimizer 會選擇最佳的執行計畫進行查詢。但是如果需要的話，我們也可以指定使用 index。

```sql
Select * from Gender with (Index(IX_Gender_GenderName))
where GenderName = 'Male'
```

![](/images/sql-opt/3-6.png)

使用 index scan 後整體的 `Estimated Subtree Cost` 是 0.0065704，比使用 table scan 的成本高。

![](/images/sql-opt/3-7.png)

## How ara data rows stored in the heap
* 資料原本以被新增的順序儲存在 heap table 中
* 當 database engine 在查詢的時候可以任意且有效的在 heap table 儲存的資料移動，無法確保資料的順序
* 為了保證回傳的資料是依照被插入的順序排列，需要在 SQL 中加入 `ORDER BY`

#### How to create and remove heaps
要新增一個 heap，在 create table 時不指定 clustered index 即可。如果 table 本來就有 clustered index，將其刪除，就會還原成 heap。

![](/images/sql-opt/3-8.jpg)

#### When to use a heap
* heap 可以用來當作大型且無順序性插入動作的臨時表格 ( staging table )。因為插入資料時沒有強制的順序，所以通常比插入有 clustered index 的表格快
* table 資料量非常小且只有幾行
* 始終透過 non-clustered indexes 來查詢資料時