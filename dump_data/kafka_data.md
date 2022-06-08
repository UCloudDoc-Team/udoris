# Kafka数据导入

登录UCloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者大数据下选择“Kafka消息队列 UKafka”，进入到[Kafka消息队列 UKafka控制台](https://console.ucloud.cn/ukafka/ukafka)下，点击**创建集群**按钮。进入创建集群页面，根据创建页提供的配置进行选择并下单。

<blockquote>
  温馨提示：
   <ol>
     <li>UKafka为付费产品，具体价格请参照创建页所展示。</li>
     <li>请确保UKafka集群和云数据仓库 UDoris集群在同一地域内（同一VPC）,如不同请使用 私有网络 VPC -> 网络互通</li>
   </ol>
</blockquote>


## 前置条件

已创建好UKafka产品并创建了Topic和Group。请参照[UKafka创建](https://docs.ucloud.cn/ukafka/kafkasinkerintro/quickstart)。

## 使用限制

1. 支持无认证的 Kafka 访问，以及通过 SSL 方式认证的 Kafka 集群。
2. 支持的消息格式如下：
   - csv 文本格式。每一个 message 为一行，且行尾**不包含**换行符。
   - Json 格式
3. 仅支持 Kafka 0.10.0.0(含) 以上版本。

## 访问SSL认证的 Kafka 集群

访问 SSL 认证的 Kafka 集群需要用户提供用于认证 Kafka Broker 公钥的证书文件（ca.pem）。如果 Kafka 集群同时开启了客户端认证，则还需提供客户端的公钥（client.pem）、密钥文件（client.key），以及密钥密码。这里所需的文件需要先通过 `CREAE FILE` 命令上传到 Plao 中，并且 catalog 名称为 `kafka`。

- 上传文件

```shell
CREATE FILE "ca.pem" PROPERTIES("url" = "https://example_url/kafka-key/ca.pem", "catalog" = "kafka");
CREATE FILE "client.key" PROPERTIES("url" = "https://example_urlkafka-key/client.key", "catalog" = "kafka");
CREATE FILE "client.pem" PROPERTIES("url" = "https://example_url/kafka-key/client.pem", "catalog" = "kafka");
```

## 操作步骤

- 通过mysql-client连接集群

```shell
mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
```

- 建库建表

```mysql
CREATE DATABASE IF NOT EXISTS ssb;
CREATE TABLE `lineorder` (
  `lo_orderkey` bigint(20) NOT NULL COMMENT "",
  `lo_linenumber` bigint(20) NOT NULL COMMENT "",
  `lo_custkey` int(11) NOT NULL COMMENT "",
  `lo_partkey` int(11) NOT NULL COMMENT "",
  `lo_suppkey` int(11) NOT NULL COMMENT "",
  `lo_orderdate` int(11) NOT NULL COMMENT "",
  `lo_orderpriority` varchar(16) NOT NULL COMMENT "",
  `lo_shippriority` int(11) NOT NULL COMMENT "",
  `lo_quantity` bigint(20) NOT NULL COMMENT "",
  `lo_extendedprice` bigint(20) NOT NULL COMMENT "",
  `lo_ordtotalprice` bigint(20) NOT NULL COMMENT "",
  `lo_discount` bigint(20) NOT NULL COMMENT "",
  `lo_revenue` bigint(20) NOT NULL COMMENT "",
  `lo_supplycost` bigint(20) NOT NULL COMMENT "",
  `lo_tax` bigint(20) NOT NULL COMMENT "",
  `lo_commitdate` bigint(20) NOT NULL COMMENT "",
  `lo_shipmode` varchar(11) NOT NULL COMMENT ""
)
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1992 VALUES [("-2147483648"), ("19930101")),
PARTITION p1993 VALUES [("19930101"), ("19940101")),
PARTITION p1994 VALUES [("19940101"), ("19950101")),
PARTITION p1995 VALUES [("19950101"), ("19960101")),
PARTITION p1996 VALUES [("19960101"), ("19970101")),
PARTITION p1997 VALUES [("19970101"), ("19980101")),
PARTITION p1998 VALUES [("19980101"), ("19990101")))
DISTRIBUTED BY HASH(`lo_orderkey`) BUCKETS 10
PROPERTIES (
"replication_num" = "3"
);
```

- 使用以下命令创建例行导入作业（访问无认证的 Kafka 集群）

```shell
CREATE ROUTINE LOAD demo.my_first_routine_load_job ON ssb.lineorder
COLUMNS TERMINATED BY ","
PROPERTIES
(
    "max_batch_interval" = "20",
    "max_batch_rows" = "300000",
    "max_batch_size" = "209715200",
)
FROM KAFKA
(
    "kafka_broker_list" = "broker1:9092,broker2:9092,broker3:9092",
    "kafka_topic" = "my_topic",
    "property.group.id" = "xxx",
    "property.client.id" = "xxx",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

- 使用以下命令创建例行导入作业（访问 SSL 认证的 Kafka 集群）

```shell
CREATE ROUTINE LOAD demo.my_first_routine_load_job ON ssb.lineorder
COLUMNS TERMINATED BY ",",
PROPERTIES
(
    "max_batch_interval" = "20",
    "max_batch_rows" = "300000",
    "max_batch_size" = "209715200",
)
FROM KAFKA
(
   "kafka_broker_list"= "broker1:9091,broker2:9091",
   "kafka_topic" = "my_topic",
   "property.security.protocol" = "ssl",
   "property.ssl.ca.location" = "FILE:ca.pem",
   "property.ssl.certificate.location" = "FILE:client.pem",
   "property.ssl.key.location" = "FILE:client.key",
   "property.ssl.key.password" = "abcdefg"
);
```

参数说明：

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| kafka_broker_list        | 以英文逗号分隔的Kafka的borker地址列表                        |
| kafka_topic              | 指定要订阅的 Kafka 的 topic                                  |
| property                 | 指定自定义kafka参数。功能等同于kafka shell中 "--property" 参数。<br />当参数的 value 为一个文件时，需要在 value 前加上关键词："FILE:"。 |
| security.protocol        | 连接方式                                                     |
| ssl.ca.location          | 如果连接方式为 SSL，需指定 CA 证书的位置                     |
| ssl.certificate.location | 如果 Kafka server 端开启了 client 认证，需要指定public key   |
| ssl.key.location         | 如果 Kafka server 端开启了 client 认证，需要指定private key  |
| ssl.key.password         | 如果 Kafka server 端开启了 client 认证，需要指定 private key 的密码 |
| kafka_default_offsets    | 指定起始 offset。默认为 `OFFSET_END`，即从末尾开始订阅       |
| group.id                 | 消费组id                                                     |
| client.id                | client 端id                                                  |
| max_batch_interval       | 每个子任务最大执行时间，单位是秒。范围为 5 到 60。默认为10。 |
| max_batch_rows           | 每个子任务最多读取的行数。必须大于等于200000。默认是200000。 |
| max_batch_size           | 每个子任务最多读取的字节数。单位是字节，范围是 100MB 到 1GB。默认是 100MB。 |

更多参数请参照官网[订阅Kafka日志](https://doris.apache.org/zh-CN/data-operate/import/import-scenes/kafka-load.html#)

- 使用mysql-client非交互模式查询验证

```shell
mysql -uroot -h<任一Frontend节点IP地址> -P9030 -p<创建集群时设置的密码> -e "select * from ssb.lineorder";
```

### 在UKafka中写入消息

请[参照UKafka](https://docs.ucloud.cn/ukafka/kafkasinkerintro/quickstart)。

