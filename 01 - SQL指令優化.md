# SQL 指令優化

## 如何建立有效率的SQL查詢
SQL指令調校過程是特別針對Where條件子句，因此條件可以決定查詢最佳話與否，後面會提供幾點參考。

#### 1. LIKE – 避免使用完用字元 (%) 開頭字串
* 當萬用字元在常數的開頭時不會使用索引進行查詢
  ```sql
  select * from STUDENT.TEST where PROGRAM_ID like '%APC%';
  ```

* 查詢條件非萬用字元開頭，就會使用索引執行查詢
  ```sql
  select * from STUDENT.TEST where PROGRAM_ID like 'APC%';
  ```

#### 2. 避免對索引值欄位進行運算或使用函數

#### 3. 避免使用OR運算子
* OR運算子在所有條件都有建可以將OR改為使用IN
* 立索引的情況下才會提升效能，但如果使用AND運算子則只需要一個條件擁有所以就可以大幅提升效能。
 
```sql
select * from STUDENT.TEST 
 where PROGRAM_DEFREE = 'B' or PROGRAM_DEFREE = 'M';
```

v.s.

```sql
select * from STUDENT.TEST 
 where PROGRAM_DEFREE in ('B', 'M');
```

#### 4. 適當使用子查詢關聯子查詢
* 可分為獨立子查詢(Uncorrelated Subquery)和關聯子查詢(Correlated Subquery)

* 關聯子查詢：
  ```sql
  select * from STUDENT.TEST
   where DEV_ID in (
     select EMP_ID from STUDENT.EMP 
     where EMP_ID = DEV_ID
  );
  ```
  * 子查詢的條件關聯到主表，所以無法單獨執行
  * 必須自外到內，先執行外面的查詢再執行子查詢


* 非關聯子查詢
  ```sql
  select * from STUDENT.TEST
   where DEV_ID in (
     select EMP_ID from STUDENT.EMP    
     where PASSWORD = '222'
  );
  ```
  * 可以獨立執行，再用執行結果與主查詢進行匹配
  * 可以用先縮減要關聯的表格資訊，再與主表對應

#### 其他查詢建議
* 盡量避免大量的排序操作，如ORDER BY、GROUP BY、HAVING等等，會讓資料庫做額外的計算，增加處理時間。
* 如果沒有需要剔除重複資料的需求，UNION ALL效能會比UNION好，因為UNION會加入類似DISTINCT的演算法。
* 盡可能在資料來源層就先過濾資料。


## 參考
* https://forum.labview360.org/t/topic/30952/1
* https://www.cc.ntu.edu.tw/chinese/epaper/0031/20141220_3109.html
