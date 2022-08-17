# SQL-Learning-Notes
<<<<<<< HEAD
* [SQL Server Tutorial for Beginners](https://www.pragimtech.com/courses/sql-server-tutorial-for-beginners/)
=======
SQL-Learning-Notes

## 查詢取得連線者帳號SQL
```sql
select inst_id, machine, count(*)
from gv$session
group by inst_id, machine
order by 1,2;
```

## Index 
* https://twgreatdaily.com/PSO_um4BMH2_cNUgTA7N.html

### Bit Tree and Bit Map
* https://vicxu.medium.com/%E7%BF%BB%E8%AD%AF-how-database-b-tree-indexing-works-8c95010e0a3a
* https://www.itread01.com/content/1543228147.html

### 其他
* https://twgreatdaily.com/PSO_um4BMH2_cNUgTA7N.html
>>>>>>> 1e8e47c08c59deee4474c03f2cd34201cf14edb9
