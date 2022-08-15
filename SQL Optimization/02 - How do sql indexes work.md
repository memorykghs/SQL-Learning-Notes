# 02 - How do sql indexes work
* 文章：https://www.pragimtech.com/blog/sql-optimization/how-do-sql-indexes-work/

本章將會介紹包含以下兩種的 index 類型：
* Clustered
* Non-clustered

## Outline
Clustered Index Structure
  * Script to create Employees table

Non-Clustered Index in SQL Server

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

## Non-Clustered Index in SQL Server
![](/images/sql-opt/2-1.jpg)

當要在 Name 欄位上建立 Non-clustered index，可以使用以下的指令。

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
