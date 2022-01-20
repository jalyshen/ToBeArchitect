# SQL 优化的方法

原文：https://blog.csdn.net/qq_38789941/article/details/83744271



1. 对查询进行优化，尽量避免全表扫描。首先应该考虑在 where 及 order by 涉及的列上简历索引

2. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描。如：

   ```sql
   select id from t where num is null
   ```

   可以在 num 上设置默认值 0， 确保表中 num 列没有 null 值，然后这样查询：

   ```sql
   select id from t where num = 0
   ```

3. 应尽量避免在 where 子句中使用 != 或者 <> 操作符，否则引擎将放弃使用索引而进行全表扫描

4. 应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全标扫描。如：

   ```sql
   select id from t where num = 10 or num = 20
   ```

   可以这样查询：

   ```sql
   select id from t where num = 10 union all 
   select id from t where num = 20
   ```

5. 慎用 in 和 not in ，否则会导致全表扫描。如：

   ```sql
   select id from t where num in (1,2,3)
   ```

   对于连续的数值，能用 between 就不要用 in ：

   ```sql
   select id from t where num between 1 and 3
   ```

6. 下面的查询也会导致全表扫描：

   ```sql
   select id from t where name like '%abc%'
   ```

7. 应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

   ```sql
   select id from t where num/2 = 100
   ```

   应该改为：

   ```sql
   select id from t where num = 100 * 2
   ```

8. 尽量避免在 where 子句中对字段进行函数操作，这也会导致索引失效而全表扫描。如：

   ```sql
   select id from t where substring(name, 1,3) = 'abc'
   ```

   可改为：

   ```sql
   select id from t where name like 'abc%'
   ```

9. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引

10. s

11. s

12. 很多时候用 exists 代替 in 是一个好的选择。如：

    ```sql
    select num from a where num in (select num fromm b);
    ```

    该用下面的语句替换：

    ```sql
    select num from a where exists(select 1 from b where num=a.num);
    ```

    

13. 并不是所有索引对查询都有效，SQL 是根据表中数据来进行查询优化的。当索引列有大量数据重复时，SQL 查询可能不会去利用索引。如一表中有字段 sex，其值 male、female 各占一半，那么即使在 sex 上建立了索引也对查询效率起不到作用

14. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。

    一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要

15. 

