# 04 - Key lookup and RID lookup
* 文章：https://www.pragimtech.com/blog/sql-optimization/key-lookup-and-rid-lookup-in-sql%20server/

Key lookup 和 RID lookup 是 SQL Server 執行計畫中常看見的操作。在這一個章節中我們將會理解這兩個概念。

## Outline
Set up Employees table
  * Script for Employees table
  * Table scan in execution plan
  * Heap table with a non-clustered index

What is RID lookup in sql server execution plans
What is key lookup in sql server execution plans
  * Create clusterd index on Id column

## Set up Employees table
同樣使用 `Employees` table 作為範例。
![](/images/sql-opt/4-1.jpg)

* 此 table 上沒有任何 clustered index
* 代表資料是無序狀態被儲存
* 當 table 沒有 clustered index 就被稱為 heap
* 一個 heap table 可能有一到多個 non-clustered indexes

#### Script for Employees table
接著對 table 寫入一些資料。

```sql
Create Table Employees
(
	Id int identity,
	[Name] nvarchar(50),
	Email nvarchar(50),
	Department nvarchar(50)
)
Go

SET NOCOUNT ON

Declare @counter int = 1

While(@counter <= 1000000)
Begin
	Declare @Name nvarchar(50) = 'ABC ' + RTRIM(@counter)
	Declare @Email nvarchar(50) = 'abc' + RTRIM(@counter) + '@pragimtech.com'
	Declare @Dept nvarchar(10) = 'Dept ' + RTRIM(@counter)

	Insert into Employees values (@Name, @Email, @Dept)

	Set @counter = @counter +1

	If(@Counter%100000 = 0)
		Print RTRIM(@Counter) + ' rows inserted'
End
```

#### Table scan in execution plan
打開 `Include Actual Execution plan` 並執行下面的 `SELECT` 語句查詢。

```sql
Select * from Employees where Name = 'ABC 932000'
```

從 execution plan 看來，是使用 table scan 進行查詢。

![](/images/sql-opt/4-2.png)

在這個情況下：
* Number of rows read = 1M
* Actual number of rows = 1

代表我們需要讀取 1M 筆資料才能找到一筆我們想要的。

* Estimated Subtree Cost = 11.4877，這是一個相對非常高的成本，所以我們接下來可以用 non-clustered index 來增加它查詢的效能。

![](/images/sql-opt/4-3.png)

#### Heap table with a non-clustered index
下列的搜尋語句使用的是 `Name` 欄位，所以我們在此欄位上加上 non-clustered index。

```sql
Select * from Employees where Name='ABC 932000'
```

建立 non-clustered index 後菜次執行查詢。

```sql
Create nonclustered index IX_Employees_Name on Employees(Name)
```

從執行計劃來看，可以發現 SQL Server 這次使用 Index Seek 來查詢資料，而不使用 Table Scan。

![](/images/sql-opt/4-4.png)

使用 non-clustered index 查詢後：
* Number of rows read 和 Actual number of rows 均為 1
* Estimated subtree cost 降低到 0.0032831

![](/images/sql-opt/4-5.png)

## What is RID lookup in sql server execution plans
RID 指的就是 Row ID。在 execution plan 中，Index Seek 也包含了 RID lookup 操作。

![](/images/sql-opt/4-6.png)

為何我們需要 Row ID look up 呢?

在使用 non-clustered index 中，我們只有 `Name` 欄位的值、以及 Row ID (RID)。但除了 `Name` 欄位值，我們還想要其他欄位的資訊 ( `EmployeeId`, `Email` 和  `Dept` )，所以 SQL Server 就可以透過 Row ID lookup 來查到對應的這筆資料的資訊。

![](/images/sql-opt/4-7.png)

接下來，我們將上面使用的 SELECT 更改一下，查詢欄位換成只針對 `Name` 欄位回傳，再次值型查詢。

```sql
Select Name from Employees where Name='ABC 932000'
```

![](/images/sql-opt/4-8.png)

可以發現：
* 在 SELECT list 中，只有顯示 `Name` 欄位以及該欄位上的 non-clustered index
* 原因是因為我們這次查詢的目的只有 `Name` 欄位，不需要同一筆資料的其他值，自然也就不用 RID lookup operation

## What is key lookup in sql server execution plans

如果 table 上沒有 clustered index，無法保證資料以任何特定順序存儲，但是如果我們創建 clustered index ( 例如在 `EmployeeId` 上 )，則表中的資料在物理上是按 `EmployeeId` 的順序儲存。我們放大檢視 tree structure 底部的 leaf node，可以看到資料是按 `EmployeeId` 列排序。

![](/images/sql-opt/4-9.jpg)

#### Create clusterd index on Id column
```sql
Create clustered index IX_Employees_Id on Employees(Id)
```

再次執行以下 SQL，可以發現使用了 Key Lookup 而不是 RID lookup。

```sql
Select * from Employees where Name = 'ABC 932000'
```

![](/images/sql-opt/4-10.png)

為什麼呢?

因為現在 `Employees` 表上同時有 clustered index 與 non-clustered indexs，所以在實體上資料是有序地被保存的。當查詢條件使用 non-clustered index 進行查詢，在該樹狀結構的 leaf node 中，除了 `Name` 欄位的值，另外還包含了 clustered index `EmployeeId` 的 pointer，SQL Server 透過 `Name` 欄位查到資訊後，可以再使用 clustered index 快速的找到該筆資料。這時候用的就是 Key Lookup 了。

![](/images/sql-opt/4-11.jpg)

再換一個 SQL 執行。

```sql
Select Id, Name from Employees where Name = 'ABC 932000'
```

這次我們在 SELECT 後面多加了 `Id` 欄位。在 execution plan 中可以發現這次我們就沒有使用 Key Lookup 了，原因是因為 `Id` 與 `Name` 欄位在 SELECT SQL 都已經被查詢出來了，所以也帶有 row locators。

![](/images/sql-opt/4-12.png)

## What is better for performance - Key lookup or RID lookup

Key Lookup 和 RID Lookup 的 `Estimated Subtree Cost` 都是一樣的 0.0065704。

![](/images/sql-opt/4-12.png)

所以哪一個效能比較好，其實要根據以下的條件：

* SQL 的語法主要要做什麼
  What the query is trying to do

* 有加上 index 的欄位以及 SELECT 出來的 list 是否包含這個欄位
  The columns you have in the index and the select list of your query

* 資料的分布
  Data distribution