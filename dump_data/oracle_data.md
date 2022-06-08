# 外部表导入数据

本文档主要介绍如何创建通过 ODBC 协议访问的外部表，以及如何导入这些外部表的数据。目前支持的数据源包括：

- MySQL
- Oracle
- PostgreSQL
- SQLServer
- Hive(1.0版本支持)

## 注意事项

- 必须保证外部数据源与UDoris 集群是可以互通，包括BE节点和外部数据源的网络是互通的。
- ODBC 外部表本质上是通过单一 ODBC 客户端访问数据源，因此并不合适一次性导入大量的数据，建议分批多次导入

## 创建外部表

- 通过mysql-client连接集群

```shell
mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
```

- 创建ODBC Resource

```mysql
CREATE EXTERNAL RESOURCE `oracle_test_odbc`
PROPERTIES (
    "type" = "odbc_catalog",
    "host" = "127.0.0.1",
    "port" = "8086",
    "user" = "oracle",
    "password" = "oracle",
    "database" = "oracle",
    "odbc_type" = "oracle",
    "driver" = "Oracle"
);
```

参数说明：

| 参数      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| type      | 必填，且必须为`odbc_catalog`。作为resource的类型标识。       |
| user      | 外部表的账号，必填                                           |
| password  | 外部表的密码，必填                                           |
| host      | 连接方式                                                     |
| port      | 外部表的连接端口，必填。                                     |
| odbc_type | 标示外部表的类型,当前doris支持`mysql`与`oracle`              |
| driver    | 标示外部表使用的driver动态库，引用该resource的ODBC外表必填，旧的mysql外表选填。 |

- 创建外部表

```mysql
CREATE EXTERNAL TABLE `ext_oracle_demo` (
  `k1` decimal(9, 3) NOT NULL COMMENT "",
  `k2` char(10) NOT NULL COMMENT "",
  `k3` datetime NOT NULL COMMENT "",
  `k5` varchar(20) NOT NULL COMMENT "",
  `k6` double NOT NULL COMMENT ""
) ENGINE=ODBC
COMMENT "ODBC"
PROPERTIES (
    "odbc_catalog_resource" = "oracle_test_odbc",
    "database" = "oracle",
    "table" = "baseall"
);
```

- 创建 Doris 表

```mysql
CREATE DATABASE IF NOT EXISTS ssb;
CREATE TABLE `doris_oralce_tbl` (
  `k1` decimal(9, 3) NOT NULL COMMENT "",
  `k2` char(10) NOT NULL COMMENT "",
  `k3` datetime NOT NULL COMMENT "",
  `k5` varchar(20) NOT NULL COMMENT "",
  `k6` double NOT NULL COMMENT ""
)
COMMENT "Doris Table"
DISTRIBUTED BY HASH(k1) BUCKETS 3;
PROPERTIES (
    "replication_num" = "3"
);
```

- 使用下列命令导入数据 (从 `ext_oracle_demo`表 导入到 `doris_oralce_tbl` 表)

```mysql
INSERT INTO ssb.doris_oralce_tbl SELECT k1,k2,k3 FROM ext_oracle_demo limit 100;
```

- 使用mysql-client非交互模式查询验证

```shell
mysql -uroot -h<任一Frontend节点IP地址> -P9030 -p<创建集群时设置的密码> -e "select * from ssb.doris_oralce_tbl";
```

其他数据源导入示例请参考[通过外部表同步数据](https://doris.apache.org/zh-CN/data-operate/import/import-scenes/external-table-load.html)

