# 插入数据（INSERT INTO）

**基本语法：**

```sql
INSERT INTO tbl SELECT ...
INSERT INTO tbl (col1, col2, ...) VALUES (1, 2, ...), (1,3, ...);
```

其中第二种命令仅用于 Demo，不要使用在测试或生产环境中。

## 导入操作及返回结果

Insert Into 命令需要通过 MySQL 协议提交，创建导入请求会同步返回导入结果。

以下是两个Insert Into的使用示例：

```sql
INSERT INTO tbl2 WITH LABEL label1 SELECT * FROM tbl3;
INSERT INTO tbl1 VALUES ("qweasdzxcqweasdzxc"), ("a");
```

> 注意：当需要使用 `CTE(Common Table Expressions)` 作为 insert 操作中的查询部分时，必须指定 `WITH LABEL` 和 column list 部分。示例：
>
> ```sql
> INSERT INTO tbl1 WITH LABEL label1
> WITH cte1 AS (SELECT * FROM tbl1), cte2 AS (SELECT * FROM tbl2)
> SELECT k1 FROM cte1 JOIN cte2 WHERE cte1.k1 = 1;
> 
> 
> INSERT INTO tbl1 (k1)
> WITH cte1 AS (SELECT * FROM tbl1), cte2 AS (SELECT * FROM tbl2)
> SELECT k1 FROM cte1 JOIN cte2 WHERE cte1.k1 = 1;
> ```

