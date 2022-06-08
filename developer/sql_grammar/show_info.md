# 查看信息

## SHOW语句

1. 获取当前集群占用的总空间大小	

```sql
curl -X GET http://fe_host:fe_http_port/api/show_data
```

2. 查看 `/statistic` 信息

```sql
GET /api/show_proc?path=/statistic

Response:
{
	"msg": "success",
	"code": 0,
	"data": [
		["10003", "default_cluster:db1", "2", "3", "3", "3", "3", "0", "0", "0"],
		["10013", "default_cluster:doris_audit_db__", "1", "4", "4", "4", "4", "0", "0", "0"],
		["Total", "2", "3", "7", "7", "7", "7", "0", "0", "0"]
	],
	"count": 0
}
```

3. 转发到 Master 执行

```sql
GET /api/show_proc?path=/statistic&forward=true

Response:
{
	"msg": "success",
	"code": 0,
	"data": [
		["10003", "default_cluster:db1", "2", "3", "3", "3", "3", "0", "0", "0"],
		["10013", "default_cluster:doris_audit_db__", "1", "4", "4", "4", "4", "0", "0", "0"],
		["Total", "2", "3", "7", "7", "7", "7", "0", "0", "0"]
	],
	"count": 0
}
```

4. 查看集群各个数据库的数据量大小

```sql
GET /api/show_meta_info?action=show_db_size

Response:
{
	"msg": "success",
	"code": 0,
	"data": {
		"default_cluster:information_schema": 0,
		"default_cluster:db1": 381
	},
	"count": 0
}
```

5. 查看FE选举组情况

```sql
GET /api/show_meta_info?action=show_ha

Response:
{
	"msg": "success",
	"code": 0,
	"data": {
		"can_read": "true",
		"role": "MASTER",
		"is_ready": "true",
		"last_checkpoint_version": "1492",
		"last_checkpoint_time": "1596465109000",
		"current_journal_id": "1595",
		"electable_nodes": "",
		"observer_nodes": "",
		"master": "10.81.85.89"
	},
	"count": 0
}
```

6.  查看数据库 my_database 中已上传的文件

```sql
SHOW FILE FROM my_database;
```

7. 该语句用于展示当前集群的配置（当前仅支持展示 FE 的配置项）

```sql
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
```

8. 查看当前FE节点的配置

```sql
ADMIN SHOW FRONTEND CONFIG;
```

9. 使用like谓词搜索当前Fe节点的配置

```sql
mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
+--------------------+-------+---------+-----------+------------+---------+
| Key                | Value | Type    | IsMutable | MasterOnly | Comment |
+--------------------+-------+---------+-----------+------------+---------+
| check_java_version | true  | boolean | false     | false      |         |
+--------------------+-------+---------+-----------+------------+---------+
1 row in set (0.01 sec)
```

10. 查看表的副本分布

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
```

11. 查看表的分区的副本分布

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
```
