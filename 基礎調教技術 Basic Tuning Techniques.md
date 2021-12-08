# 基礎調教技術 Basic Tuning Techniques

## Case 1 - Table Design
```sql
select TWON_NAME from CUSTOMER
where ZIP_CODE_1 = substr(ADDRESS, 1, 2);
```
當條件中有使用到函式，都有可能讓 Optimizer 忽略掉在該列上使用索引可能性。

## Case 2 - Index Usage
索引可以用於三種類型的條件：
1. 相等搜尋 [A]
2. 無界範圍 [B、C、D]
3. 有界範圍 [E]

```sql
--[A]
select EMP_NAME from EMPLOYEE where SALARY = 30000;

--[B]
select EMP_NAME from EMPLOYEE where SALARY   30000;

--[C]
select EMP_NAME from EMPLOYEE where SALARY < 30000;

--[D]
select EMP_NAME from EMPLOYEE where SALARY > 30000;

--[E]
select EMP_NAME from EMPLOYEE where SALARY between 30000 and 40000;
```

不過當 SQL 條件存在 `NOT EQUAL <>` 運算符，則不會使用索引。不過要注意的是，空值 `null` 不會有索引。

## Case 3 - Transform Index
如果很常使用某個欄位運算後的值作為查詢條件，那麼應該針對該運算後的值建立索引，而不是單純建在該欄位上。
```sql
select EMP_NAME from EMPLOYEE where SALARY * 12 = 360000;
```

## Case 4 - Data Type Mismatch
當查詢條件中具有強制隱式的數據類型轉換，不會使用索引。應該比免這種混和模式的 SQL 表達式，以及小心處理隱式類型的轉換。

像下面的例子，原本的欄位型別為字串，但查詢時將它轉為數字查詢，就有使用到隱式數據類型轉換。
```sql
select EMP_NAME from EMPLOYEE where EMP_ID = 12345;
```

## Case 5 - 調整 Order By 子句
當 Order By 條件的欄位是沒有 Index 的時候，執行過程可能需要去執行下面的 Execution Plan 中的 `SORT ORDER BY`，然而正確的使用 Order By 搭配 Index 的話，其實是不需要下的。因為有 Index 的欄位應該要包含該欄位的順序，不需要重新排。
```
---------------------------------------------------------------
|Id | Operation                    | Name       | Rows | Cost |
---------------------------------------------------------------
| 0 | select STATEMENT             |            |  320 |   18 |
| 1 |  SORT ORDER BY               |            |  320 |   18 |
| 2 |   TABLE ACCESS BY INDEX ROWID| SALES      |  320 |   17 |
|*3 |    INDEX RANGE SCAN          | SALES_DATE |  320 |    3 |
---------------------------------------------------------------
```

來看幾個例子：

```sql
--[A]
select CUST_FIRST_NAME , CUST_LAST_NAME
from CUSTOMERS
order by CUST_CREDIT_LIMIT;
```
首先，[A] 中的語句使用非 Index 欄位進行 Order By，執行計畫中需要針對該欄位進行排序。

```sql
--[B] 
select CUST_FIRST_NAME, CUST_LAST_NAME
from CUSTOMERS
order by CUST_ID;
```
將 Order By 的欄位換成 PK ( 有 Index 的欄位 ) CUST_ID，執行 SQL 時同樣要對該欄位排序。

```sql
--[C] 
select CUST_FIRST_NAME, CUST_LAST_NAME, CUST_CITY
from CUSTOMERS
where CUST_CITY = 'Paris'
order by CUST_ID;
```
加上 where 條件，仍然沒有使用已經建立的 Index。

```sql
--[D] 
select CUST_FIRST_NAME, CUST_LAST_NAME, CUST_CITY
from CUSTOMERS
where CUST_ID < 200
order by CUST_ID;
```
將 where 條件換成 PK 欄位，才有使用到 Index。

> **結論**
如果使用有 Index 且 not nullable 的欄位 Order By，花費的成本較低。

## Case 6 - Correlation Subquery
如果查詢條件包括子查詢，那麼下面的 EMPLOYEE 表格在查詢的時候，每一行都會執行一次子查詢。

```sql
SELECT DEPARTMENT_ID, last_name, salary 
FROM employees e1 
WHERE salary > (SELECT AVG(salary) 
                FROM employees e2
                WHERE e1.DEPARTMENT_ID = e2.DEPARTMENT_ID
                GROUP BY e2.DEPARTMENT_ID) 
ORDER BY DEPARTMENT_ID
```

將子查詢的部分改為以 `Join` 的方式可以減少操作的次數。
```sql
SELECT employee_id, last_name, salary
FROM employees e1
          , (SELECT avg(salary) avg_sal, DEPARTMENT_ID
             FROM employees
             GROUP BY DEPARTMENT_ID) a
WHERE e1.salary > a.avg_sal
AND e1.DEPARTMENT_ID = a.DEPARTMENT_ID
```

不過目前的 Oracle 遇到子查詢，operator 會自動轉為 `Join` 執行。

## Case 7 - Union & Union All
無論是否存在索引，`Union` 運算子會對查詢結果無條件的進行排序，並篩選出不重複的結果，如果要使用 `Union`，可以依照需求建立欄位的索引。

但使用用 `Union All` 運算子十，operator 不會對查詢結果進行篩選和排序，有可能導致結果與預期的不相符。
```sql
select count(*)
from (
select *
  from CUSTOMERS
 where CUST_FIRST_NAME = 'Max'
union 
select *
  from CUSTOMERS
 where CUST_LAST_NAME = 'Colven'); -> 127 rows
 
select count(*)
from (
select *
  from CUSTOMERS
 where CUST_FIRST_NAME = 'Max'
union all
select *
  from CUSTOMERS
 where CUST_LAST_NAME = 'Colven'); -> 143 rows
```

## Case 8 - 避免使用 HAVING 運算子
從執行計畫上來看，使用 `Having` 運算子的執行計畫並沒有使用到索引，因為 `Having` 的執行順序在 `Group By` 之後，且 `Having` 運算子後通常會搭配函式如 `COUNT`、`SUM`、`MAX`、`MIN` 或是 `AVG`。
```sql
select CUST_ID, avg(CUST_CREDIT_LIMIT)
 from CUSTOMERS
 group by CUST_CITY
 having CUST_CITY = 'paris';
```

改為使用等號或是才有機會用到索引。
```sql
select CUST_ID, avg(CUST_CREDIT_LIMIT)
 from CUSTOMERS
 group by CUST_CITY
 having CUST_CITY = 'paris';
```

## Case 9 - Like '%String'
當使用 `Like` 搭配 `%` 查詢，執行計畫可能不會使用索引。
```sql
select PROGRAM_ID from STUDENT.TEST where 
```

## Case - 複合 PK 索引欄位的順序


## 參考
* https://use-the-index-luke.com/sql/sorting-grouping/indexed-order-by