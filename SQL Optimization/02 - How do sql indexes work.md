# 02 - How do sql indexes work
* 文章：https://www.pragimtech.com/blog/sql-optimization/how-do-sql-indexes-work/

本章將會介紹包含以下兩種的 index 類型：
* Clustered
* Non-clustered

## Outline
Clustered Index Structure
  * Script to create Employees table

Clustered Index Scan
Non-Clustered Index in SQL Server
  * Non-clustered and clustered index in action

## Clustered Index Structure
如同前一章提到的，SQL Server 預設會以 clustered key 的結構去儲存資料。

#### Script to create Employees table
```sql
Create Table Employees
(
	Id int primary key identity,
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

然後在 SQL Server Management Studio 中使用 Include Actual Execution Plan 並執行以下查詢。
```sql
Select * from Employees where Id = 932000
```

其執行計畫如下：
![](/images/sql-opt/2-6.png)

* 執行計畫中使用 Clustered Index Seek，代表 database engine 使用 clustered index 去搜尋 `EmployeeId`，其中：
  * 讀取的 rows 數量 = 1
  * 實際找到的 rows 數量 = 1 ( 查詢到的結果筆數 )

讀取的 rows 數量代表 SQL Server 要找到指定資料需要讀取的筆數。在此範例中，因為 `EmployeeId` 是 PK，具有唯一性，所以我們會預期它實際上也只需要讀取一次就能夠找到需要的資料。

## Clustered Index Scan
由於 `EmployeeId` 上有 clustered index，所以當使用此欄位作為條件查詢時，SQL Server 可以快速地找到資料。但是當使用 `Name` 欄位進行搜尋呢? `Name` 欄位上沒有叢集索引，所以 SQL Server 必須一筆一筆資料進行查詢。

```sql
Select * from Employees Where Name = 'ABC 932000'
```

上述 SQL 的 execution plan 如下：
![](/images/sql-opt/2-7.png)

* 使用 `Clustered Index Scan` ( 而不是 `Clustered Index Seek` )
* 讀取的 rows 數量 = 1000000
* 實際需要讀取到的 rows 筆數 = 1 ( 因為只找到一筆資料 )

> 由此可知，Index Scan 的效能其實非常低

## Non-Clustered Index in SQL Server
![](/images/sql-opt/2-1.jpg)

當要在 `Name` 欄位上建立 Non-clustered index，可以使用以下的指令。

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Name
ON [dbo].[Employees] ([Name])
```

![](/images/sql-opt/2-2.png)

* 在 non-clustered index 中，並不會包含 table 中的資料。只會儲存 key value 及 row locators
* 在這個例子中，key value 就是 Name 欄位的值，它會以自然字母排序 ( alphabetical order ) 被儲存
* 在樹狀結構最底端的 row locators 會包含：
  * Employee Names
  * 該筆資料的 cluster key ( `EmployeeId` )

再次執行查詢指令。
```sql
Select * from Employees Where Name = 'ABC 932000'
```

此執行計畫由右 → 左，由上 → 下看。
![](/images/sql-opt/2-3.png)

#### Non-clustered and clustered index in action
當執行以下的 SQL 時：
```sql
Select * from Employees where Name='David'
```

![](/images/sql-opt/2-4.png)

* SQL Server 使用 non-clustered index 去搜尋 `Name` 欄位
* 在 non-clustered 中除了紀錄 `Name` 欄位的值，也會記錄 cluster key ( `EmployeeId` )

`Estimated Subtree Cost` 上的有呈現使用非 index 與 non-clustered index 的成本差異。

![](/images/sql-opt/2-5.png)