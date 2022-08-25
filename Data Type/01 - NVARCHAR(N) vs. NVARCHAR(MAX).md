# 01 - NVARCHAR(N) vs. NVARCHAR(MAX) Performance in MS-SQL Server
* 文章：https://hungdoan.com/2017/04/13/nvarcharn-vs-nvarcharmax-performance-in-ms-sql-server/

讀完這篇將會了解：
* Why Don't We Always Use NVARCHAR(MAX)?
* Are NVARCHAR(N) and NVARCHAR(MAX) Different in Performance?

* Are NVARCHAR(N) and NVARCHAR(MAX) Different in Storage Size?

## Outline
What is nvarchar(n)?

## What is nvarchar(n)
首先，什麼是 nvarchar(n)?

> **nvarchar [ ( n | max ) ]**
是長度可變動 (Variable-length) 的 Unicode 字串格式資料。`n` 則是定用來定義長度，`n` 的值介於 1 ~ 4000。

> * `max` 指最大可以儲存到 `2^31-1` bytes ( 約 2 GB )。
> * 實際儲存的大小是：輸入長度 * 2 bytes 後再加上 2 bytes
> * `nvarchar` 的 ISO 同義詞是 `national char varying` 和 `national character varying`。

(From：[Microsoft Doc - nchar and nvarchar](https://docs.microsoft.com/en-us/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?view=sql-server-ver16))

nvarchar(n) 中的 `n` 定義了從 1 到 4000 的字符串長度。如果字符串長度超過 4000，則可以改為定義 nvarchar(max)。

## What is different between nvarchar(7) and nvarchar(4000)?
假設，儲存一個長度為 7 的字串："fortune"。那麼，nvarchar(7) 和 nvarchar(4000) 有什麼區別？

答案是：**nvarchar(7) 和 nvarchar(4000) 在效能和存儲的大小方面沒有區別**。

那在設計資料庫時，`n` 越大越好嗎? ( 反正不夠就往上擴啊xDD )

這個部分，可以分成兩個面向：
* 字串的實際大小會影響索引的效能，如果超過 900 ( 450 個 Unicode 字元 )，該欄位就不能被建立索引
* nvarchar(7) 和 nvarchar(4000)，比起 nvarchar(max) 是存在效能差異的

原因是因為在 MS-SQL 中他們被儲存的方式是不同的。在 MS-SQL，所有資料會被儲存在以 8-KB 為單位的 Pages 中。
![](/images/data-type/1-1.png)

在上圖中可以看到有三種類型的 allocation Units ( 也被稱為 data clusters )。

* **Data (IN_ROW_DATA)**：Data ( index rows ) 包含了所有資料列的資料，除了大型的資料對象 ( LOB ) 除外
* **LOB (LOB_DATA)**：儲存在以下一種或多種的資料型態：text, ntext, image, xml, varchar(max), nvarchar(max), varbinary(max), or CLR 使用者定義類型 ( CLR UDT ) 資料
* **Row overflow (ROW_OVERFLOW_DATA)**：儲存在以下資料類型：varchar, nvarchar, varbinary, or sql_variant 且可變動資料長度超過 8060 字元大小限制的資料

如果我們使用 nvarchar(max)，數據將存儲在 **LOB_DATA** 而不是 **IN_ROW_DATA** 或 **ROW_OVERFLOW_DATA**。

結論：
* 一般情況下，所有的資料被儲存在 **IN_ROW_DATA** 並且在執行上有較好的效能。
* 但如果一行資料超過 8KB 或資料型別是 LOB (nvarchar(max))，MS-SQL 就會將資料列拆分到另外一個區域 ( LOB_DATA 或 ROW_OVERFLOW_DATA )
* 然後在 IN_ROW_DATA 留下一個指向 LOB_DATA/ROW_OVERFLOW_DATA 的 pointer。

以上就是為什麼 nvarchar(max) 的效能會較 nvarchar(n) 來的差。

## Conclusion
##### WHY DON'T WE ALWAYS USE NVARCHAR(MAX)?
* nvarchar(max) 較  nvarchar(n) 效能差
* 型別為 nvarchar(max) 無法被加上索引

##### ARE NVARCHAR(N) AND NVARCHAR(MAX) DIFFERENT IN PERFORMANCE?
是ㄉ，nvarchar(max) 較 nvarchar(n) 慢

##### ARE NVARCHAR(N) AND NVARCHAR(MAX) DIFFERENT IN STORAGE SIZE?
nvarchar(max) 會因為有紀錄指向的 pointer 而多花幾個 bytes，但這不是什麼值得擔心的大問題。因為如果一筆資料大於 8KB 的話同樣也會撥出一個 pointer。
