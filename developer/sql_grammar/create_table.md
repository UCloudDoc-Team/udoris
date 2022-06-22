# 建表（Create Table）

- ### 创建表

**语法：**

```sql
CREATE TABLE [IF NOT EXISTS] [database.]table
(
    column_definition_list,
    [index_definition_list]
)
[engine_type]
[keys_type]
[table_comment]
[partition_info]
distribution_info
[rollup_list]
[properties]
[extra_properties]
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
        <td><code>database</code></td>
        <td>数据库的名称。</td>
    </tr>
    <tr>
        <td><code>column_definition_list</code></td>
        <td>列定义列表。</td>
    </tr>
    <tr>
        <td><code>column_definition</code></td>
        <td>列定义。</td>
    </tr>
    <tr>
        <td><code>column_type</code></td>
        <td>列类型。云数据仓库UDoris支持的数据类型，
            请参见<a title="" href="/udoris/developer/data_type">数据类型</a></span>。
        </td>
    </tr>
    <tr>
        <td>
            <code>aggr_type</code>
        </td>
        <td>聚合类型。云数据库UDoris支持的聚合类型，请参见
            <a href="/udoris/developer/data_type">聚合类型</a></span>。
    	</td>
</tr>
<tr>
    <td><code>index_definition</code></td>
    <td>索引定义。</td>
</tr>
<tr>
    <td><code>engine_type</code></td>
    <td>表引擎类型，本文档中类型皆为 OLAP。</td>
</tr>
<tr>
    <td><code>key_type</code></td>
    <td>数据模型。
        <ul>
            <li>DUPLICATE KEY（默认）：其后指定的列为排序列。</li>
            <li>AGGREGATE KEY：其后指定的列为维度列。</li>
            <li>UNIQUE KEY：其后指定的列为主键列。</li>
        </ul>
    </td>
</tr>
<tr>
    <td>
        <code>table_comment</code>
    </td>
    <td>表注释。</td>
</tr>
<tr>
    <td><code>partition_desc</code></td>
    <td>分区信息。</td>
</tr>
<tr>
    <td><code>distribution_desc</code></td>
    <td>定义数据分桶方式。</td>
</tr>
<tr>
    <td><code>rollup_list</code></td>
    <td>建表的同时可以创建多个物化视图。</td>
</tr>
<tr>
    <td><code>properties</code></td>
    <td>设置表属性。目前支持以下属性。
        <ul>
            <li>replication_num：副本数。默认副本数为3。</li>
            <li>replication_allocation：根据Tag设置副本分布情况。该属性可以完全覆盖 replication_num 属性的功能。</li>
            <li>storage_medium/storage_cooldown_time：数据存储介质。</li>
            <li>colocate_with：当需要使用Colocation Join功能时，使用这个参数设置 Colocation Group。</li>
            <li>bloom_filter_columns：用户指定需要添加Bloom Filter索引的列名称列表。各个列的 Bloom Filter 索引是独立的，并不是组合索引。</li>
            <li>in_memory：Doris 是没有内存表的概念。</li>
          	<li>function_column.sequence_type：当使用UNIQUE KEY模型时，可以指定一个sequence列，当KEY列相同时，将按照sequence列进行REPLACE(较大值替换较小值，否则无法替换。</li>
          	<li>dynamic_partition.enable：用于指定表级别的动态分区功能是否开启。默认为 true。</li>
          	<li>dynamic_partition.time_unit：用于指定动态添加分区的时间单位，可选择为DAY（天），WEEK(周)，MONTH（月），HOUR（时）。</li>
         		<li>dynamic_partition.start：用于指定向前删除多少个分区。值必须小于0。默认为 Integer.MIN_VALUE。</li>
         		<li>dynamic_partition.end：用于指定提前创建的分区数量。值必须大于0。</li>
          	<li>dynamic_partition.prefix：用于指定创建的分区名前缀，例如分区名前缀为p，则自动创建分区名为p20200108。</li>
            <li>dynamic_partition.buckets: 用于指定自动创建的分区分桶数量。</li>
            <li>dynamic_partition.create_history_partition: 是否创建历史分区。</li>
            <li>dynamic_partition.history_partition_num: 指定创建历史分区的数量。</li>
            <li>dynamic_partition.reserved_history_periods: 用于指定保留的历史分区的时间段。</li>
            <li>data_sort.sort_type: 数据排序使用的方法，目前支持两种：lexical/z-order，默认是lexical。</li>
            <li>data_sort.col_num: 数据排序使用的列数，取最前面几列，不能超过总的key列数。</li>
        </ul>
    </td>
</tr>
</tbody>
</table>

**示例：创建一个明细模型的表，分区，指定排序列，设置副本数为3**

```sql
CREATE TABLE example_db.table_hash
(
    k1 DATE,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    k3 CHAR(10) COMMENT "string column",
    k4 INT NOT NULL DEFAULT "1" COMMENT "int column"
)
DUPLICATE KEY(k1, k2)
COMMENT "my first table"
PARTITION BY RANGE(k1)
(
    PARTITION p1 VALUES LESS THAN ("2020-02-01"),
    PARTITION p1 VALUES LESS THAN ("2020-03-01"),
    PARTITION p1 VALUES LESS THAN ("2020-04-01")
)
DISTRIBUTED BY HASH(k1) BUCKETS 32
PROPERTIES (
    "replication_num" = "3"
);
```

**示例：创建一个聚合模型表，使用固定范围分区描述**

```sql
CREATE TABLE table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048) REPLACE,
    v2 INT SUM DEFAULT "1"
)
AGGREGATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1, k2, k3)
(
    PARTITION p1 VALUES [("2014-01-01", "10", "200"), ("2014-01-01", "20", "300")),
    PARTITION p2 VALUES [("2014-06-01", "100", "200"), ("2014-07-01", "100", "300"))
)
DISTRIBUTED BY HASH(k2) BUCKETS 32
```

**示例：创建两张同一个 Colocation Group 自维护的表。**

```sql
CREATE TABLE t1 (
    id int(11) COMMENT "",
    value varchar(8) COMMENT ""
)
DUPLICATE KEY(id)
DISTRIBUTED BY HASH(id) BUCKETS 10
PROPERTIES (
    "colocate_with" = "group1"
);

CREATE TABLE t2 (
    id int(11) COMMENT "",
    value1 varchar(8) COMMENT "",
    value2 varchar(8) COMMENT ""
)
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`) BUCKETS 10
PROPERTIES (
    "colocate_with" = "group1"
);
```

**示例：创建一个动态分区表。**

```sql
CREATE TABLE example_db.dynamic_partition
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1) ()
DISTRIBUTED BY HASH(k2) BUCKETS 32
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "32" 
);
```



- ### 创建外部表
- 在Doris创建外部表的目的是可以通过Doris访问外部数据库。而Doris本身并不维护、存储任何外部数据。

主要通过 ENGINE 类型来标识是哪种类型的外部表，目前可选 ODBC、HIVE、ICEBERG 、HUDI。

**示例：通过ODBC_Resource来创建ODBC外表。**

```sql
CREATE EXTERNAL RESOURCE `oracle_odbc`
PROPERTIES (
"type" = "odbc_catalog",
"host" = "192.168.0.1",
"port" = "8086",
"user" = "test",
"password" = "test",
"database" = "test",
"odbc_type" = "oracle",
"driver" = "Oracle 19 ODBC driver"
);
     
CREATE EXTERNAL TABLE `baseall_oracle` (
  `k1` decimal(9, 3) NOT NULL COMMENT "",
  `k2` char(10) NOT NULL COMMENT "",
  `k3` datetime NOT NULL COMMENT "",
  `k5` varchar(20) NOT NULL COMMENT "",
  `k6` double NOT NULL COMMENT ""
) ENGINE=ODBC
COMMENT "ODBC"
PROPERTIES (
"odbc_catalog_resource" = "oracle_odbc",
"database" = "test",
"table" = "baseall"
);
```

**示例：创建一个hive外部表。**

```sql
CREATE TABLE example_db.table_hive
(
  k1 TINYINT,
  k2 VARCHAR(50),
  v INT
)
ENGINE=hive
PROPERTIES
(
  "database" = "hive_db_name",
  "table" = "hive_table_name",
  "hive.metastore.uris" = "thrift://127.0.0.1:9083"
);
```

**示例：创建一个 Iceberg 外表。**

```sql
CREATE TABLE example_db.t_iceberg 
ENGINE=ICEBERG
PROPERTIES (
	"iceberg.database" = "iceberg_db",
	"iceberg.table" = "iceberg_table",
	"iceberg.hive.metastore.uris"  =  "thrift://127.0.0.1:9083",
	"iceberg.catalog.type"  =  "HIVE_CATALOG"
);
```

**示例：创建一个 Hudi 外表。**

```sql
CREATE TABLE example_db.t_hudi
ENGINE=HUDI
PROPERTIES (
"hudi.database" = "hudi_db_in_hive_metastore",
"hudi.table" = "hudi_table_in_hive_metastore",
"hudi.hive.metastore.uris" = "thrift://127.0.0.1:9083"
);
```

**创建表时，关于 Partition 和 Bucket 的数量和数据量的建议。**

- 一个表的 Tablet 总数量等于 (Partition num * Bucket num)。
- 一个表的 Tablet 数量，在不考虑扩容的情况下，推荐略多于整个集群的磁盘数量。
- 单个 Tablet 的数据量理论上没有上下界，但建议在 1G - 10G 的范围内。如果单个 Tablet 数据量过小，则数据的聚合效果不佳，且元数据管理压力大。如果数据量过大，则不利于副本的迁移、补齐，且会增加 Schema Change 或者 Rollup 操作失败重试的代价（这些操作失败重试的粒度是 Tablet）。
- 当 Tablet 的数据量原则和数量原则冲突时，建议优先考虑数据量原则。
- 在建表时，每个分区的 Bucket 数量统一指定。但是在动态增加分区时（`ADD PARTITION`），可以单独指定新分区的 Bucket 数量。可以利用这个功能方便的应对数据缩小或膨胀。
- 一个 Partition 的 Bucket 数量一旦指定，不可更改。所以在确定 Bucket 数量时，需要预先考虑集群扩容的情况。比如当前只有 3 台 host，每台 host 有 1 块盘。如果 Bucket 的数量只设置为 3 或更小，那么后期即使再增加机器，也不能提高并发度。
- 举一些例子：假设在有10台BE，每台BE一块磁盘的情况下。如果一个表总大小为 500MB，则可以考虑4-8个分片。5GB：8-16个分片。50GB：32个分片。500GB：建议分区，每个分区大小在 50GB 左右，每个分区16-32个分片。5TB：建议分区，每个分区大小在 50GB 左右，每个分区16-32个分片

**官网参考文档**

- 创建表的更多信息，请参见[CREATE TABLE](https://doris.apache.org/zh-CN/docs/sql-manual/sql-reference/Data-Definition-Statements/Create/CREATE-TABLE.html#create-table)。
- 创建外部表，请参见[CREATE-EXTERNAL-TABLE](https://doris.apache.org/zh-CN/docs/sql-manual/sql-reference/Data-Definition-Statements/Create/CREATE-EXTERNAL-TABLE.html#create-external-table)。
