# 创建视图（Create View）

**基本语法：**

```sql
CREATE VIEW [IF NOT EXISTS]
 [db_name.]view_name
 (column1[ COMMENT "col comment"][, column2, ...])
AS query_stmt
```

**参数说明：**

<table>
    <thead>
        <tr>
            <th>参数</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>db</code></td>
        <td>数据库的名称。</td>
    </tr>
    <tr>
        <td><code>view_name</code></td>
        <td>视图名。</td>
    </tr>
    <tr>
        <td><code>query_stmt</code></td>
        <td>任意支持的SQL</td>
    </tr>
    </tbody>
</table>

**示例：**

1. 在 example_db 上创建视图 example_view。

```sql
CREATE VIEW example_db.example_view (k1, k2, k3, v1)
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

2. 创建一个包含 comment 的 view。

```sql
CREATE VIEW example_db.example_view
(
    k1 COMMENT "first key",
    k2 COMMENT "second key",
    k3 COMMENT "third key",
    v1 COMMENT "first value"
)
COMMENT "my first view"
AS
SELECT c1 as k1, k2, k3, SUM(v1) FROM example_table
WHERE k1 = 20160112 GROUP BY k1,k2,k3;
```

官网参考文档

创建视图的更多信息，请参见[Create View](https://doris.apache.org/docs/sql-manual/sql-reference/Data-Definition-Statements/Create/CREATE-VIEW)。