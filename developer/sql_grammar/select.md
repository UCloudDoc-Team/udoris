# 查询（SELECT）

**基本语法：**

```sql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    select_expr [, select_expr ...]
    [FROM table_references
      [PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [INTO OUTFILE 'file_name']
```

**简单查询语句示例：**

```sql
select Name from student where age in (18,20,25);
```

1. GROUP BY 示例

   ```sql
   --查询tb_book表，按照type分组，求每类图书的平均价格,
   select type,avg(price) from tb_book group by type;
   ```

2. DISTINCT 使用

   ```sql
   --查询tb_book表，除去重复的type数据
   select distinct type from tb_book;
   ```

3. ORDER BY 示例

   对查询结果进行升序（默认）或降序（DESC）排列。升序NULL在最前面，降序NULL在最后面

   ```sql
   --查询tb_book表中的所有记录，按照id降序排列，显示三条记录
   select * from tb_book order by id desc limit 3;
   ```

4. LIKE模糊查询

   可实现模糊查询，它有两种通配符：`%`和`_`，`%`可以匹配一个或多个字符，`_`可以匹配一个字符

   ```sql
   --查找所有第二个字符是h的图书
   select * from tb_book where name like('_h%');
   ```

5. LIMIT限定结果行数

   ```sql
   ql--1.降序显示3条记录
   select * from tb_book order by price desc limit 3;
   
   --2.从id=1显示4条记录
   select * from tb_book where id limit 1,4;
   ```

6. CONCAT联合多列

   ```sql
   --把name和price合并成一个新的字符串输出
   select id,concat(name,":",price) as info,type from tb_book;
   ```

7. 使用函数和表达式

   ```sql
   --计算tb_book表中各类图书的总价格
   select sum(price) as total,type from tb_book group by type;
   --price打八折
   select *,(price * 0.8) as "八折" from tb_book;
   ```

## JOIN语法

**基本语法：**

```sql
JION
table_references:
    table_reference [, table_reference] …
table_reference:
    table_factor
  | join_table
table_factor:
    tbl_name [[AS] alias]
        [{USE|IGNORE|FORCE} INDEX (key_list)]
  | ( table_references )
  | { OJ table_reference LEFT OUTER JOIN table_reference
        ON conditional_expr }
join_table:
    table_reference [INNER | CROSS] JOIN table_factor [join_condition]
  | table_reference STRAIGHT_JOIN table_factor
  | table_reference STRAIGHT_JOIN table_factor ON condition
  | table_reference LEFT [OUTER] JOIN table_reference join_condition
  | table_reference NATURAL [LEFT [OUTER]] JOIN table_factor
  | table_reference RIGHT [OUTER] JOIN table_reference join_condition
  | table_reference NATURAL [RIGHT [OUTER]] JOIN table_factor
join_condition:
    ON conditional_expr
```

**JOIN语句示例：**

```sql
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
                 ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```

1. INNER JOIN

   ```sql
   SELECT t1.name, t2.salary
     FROM employee AS t1 INNER JOIN info AS t2 ON t1.name = t2.name;
   
   SELECT t1.name, t2.salary
     FROM employee t1 INNER JOIN info t2 ON t1.name = t2.name;
   ```

2. LEFT JOIN

   ```sql
   SELECT left_tbl.*
     FROM left_tbl LEFT JOIN right_tbl ON left_tbl.id = right_tbl.id
     WHERE right_tbl.id IS NULL;
   ```

3. RIGHT JOIN

   ```sql
   mysql> SELECT * FROM t1 RIGHT JOIN t2 ON (t1.a = t2.a);
   +------+------+------+------+
   | a    | b    | a    | c    |
   +------+------+------+------+
   |    2 | y    |    2 | z    |
   | NULL | NULL |    3 | w    |
   +------+------+------+------+
   ```

## **UNION语法：**

**基本语法：**

```sql
SELECT ...
UNION [ALL| DISTINCT] SELECT ......
[UNION [ALL| DISTINCT] SELECT ...]
```

`UNION` 用于将多个 `SELECT` 语句 的结果组合 到单个结果集中。

第一个 `SELECT` 语句中的列名称用作返回结果的列名称。 在每个 `SELECT`语句的 相应位置列出的选定列 应具有相同的数据类型。 （例如，第一个语句选择的第一列应该与其他语句选择的第一列具有相同的类型。）

默认行为 `UNION`是从结果中删除重复的行。 可选 `DISTINCT` 关键字除了默认值之外没有任何效果，因为它还指定了重复行删除。 使用可选 `ALL` 关键字，不会发生重复行删除，结果包括所有 `SELECT` 语句中的 所有匹配行

**UNION 示例**

```sql
SELECT a FROM t1 WHERE a = 10 AND B = 1 ORDER by LIMIT 10
UNION
SELECT a FROM t2 WHERE a = 11 AND B = 2 ORDER by LIMIT 10;
```

## **WITH语句**：

**基本语法：**

```sql
WITH
  cte1 AS（SELECT a，b FROM table1），
  cte2 AS（SELECT c，d FROM table2）
SELECT b，d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```

在包含该 `WITH`子句 的语句中 ，可以引用每个 CTE 名称以访问相应的 CTE 结果集。

CTE 名称可以在其他 CTE 中引用，从而可以基于其他 CTE 定义 CTE。

CTE 可以引用自身来定义递归 CTE 。 递归 CTE 的常见应用包括分层或树状结构数据的序列生成和遍历。

**WITH 子句示例**

```sql
WITH cte AS
(
  SELECT 1 AS col1, 2 AS col2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```