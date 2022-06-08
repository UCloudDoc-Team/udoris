# 多表测试

## 新建云数据仓库UDoris

请参阅 [快速上手](/udoris/gettingstart)

## 连接云数据仓库UDoris

请参阅 **操作指南->[连接集群](/udoris/operation_guide/connect_cluster)**

## 执行建库建表

**DDL语句如下：**

```sql
#创建数据库
CREATE DATABASE IF NOT EXISTS ssb_test ;
#创建 customer
CREATE TABLE IF NOT EXISTS ssb_test.customer  
(
  `c_custkey` int(11) NOT NULL COMMENT "",
  `c_name` varchar(26) NOT NULL COMMENT "",
  `c_address` varchar(41) NOT NULL COMMENT "",
  `c_city` varchar(11) NOT NULL COMMENT "",
  `c_nation` varchar(16) NOT NULL COMMENT "",
  `c_region` varchar(13) NOT NULL COMMENT "",
  `c_phone` varchar(16) NOT NULL COMMENT "",
  `c_mktsegment` varchar(11) NOT NULL COMMENT ""
)
DISTRIBUTED BY HASH(`c_custkey`) BUCKETS 10
PROPERTIES (
"replication_num" = "3"
);
#创建 lineorder
CREATE TABLE IF NOT EXISTS ssb_test.lineorder
(
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
#创建 part
CREATE TABLE IF NOT EXISTS ssb_test.part 
(
  `p_partkey` int(11) NOT NULL COMMENT "",
  `p_name` varchar(23) NOT NULL COMMENT "",
  `p_mfgr` varchar(7) NOT NULL COMMENT "",
  `p_category` varchar(8) NOT NULL COMMENT "",
  `p_brand` varchar(10) NOT NULL COMMENT "",
  `p_color` varchar(12) NOT NULL COMMENT "",
  `p_type` varchar(26) NOT NULL COMMENT "",
  `p_size` int(11) NOT NULL COMMENT "",
  `p_container` varchar(11) NOT NULL COMMENT ""
)
DISTRIBUTED BY HASH(`p_partkey`) BUCKETS 10
PROPERTIES (
"replication_num" = "3"
);
#创建 supplier
CREATE TABLE IF NOT EXISTS ssb_test.supplier 
(
  `s_suppkey` int(11) NOT NULL COMMENT "",
  `s_name` varchar(26) NOT NULL COMMENT "",
  `s_address` varchar(26) NOT NULL COMMENT "",
  `s_city` varchar(11) NOT NULL COMMENT "",
  `s_nation` varchar(16) NOT NULL COMMENT "",
  `s_region` varchar(13) NOT NULL COMMENT "",
  `s_phone` varchar(16) NOT NULL COMMENT ""
)
DISTRIBUTED BY HASH(`s_suppkey`) BUCKETS 10
PROPERTIES (
"replication_num" = "1"
);
#创建 dates
CREATE TABLE IF NOT EXISTS ssb_test.date
(
  `d_datekey` int(11) NOT NULL COMMENT "",
  `d_date` varchar(20) NOT NULL COMMENT "",
  `d_dayofweek` varchar(10) NOT NULL COMMENT "",
  `d_month` varchar(11) NOT NULL COMMENT "",
  `d_year` int(11) NOT NULL COMMENT "",
  `d_yearmonthnum` int(11) NOT NULL COMMENT "",
  `d_yearmonth` varchar(9) NOT NULL COMMENT "",
  `d_daynuminweek` int(11) NOT NULL COMMENT "",
  `d_daynuminmonth` int(11) NOT NULL COMMENT "",
  `d_daynuminyear` int(11) NOT NULL COMMENT "",
  `d_monthnuminyear` int(11) NOT NULL COMMENT "",
  `d_weeknuminyear` int(11) NOT NULL COMMENT "",
  `d_sellingseason` varchar(14) NOT NULL COMMENT "",
  `d_lastdayinweekfl` int(11) NOT NULL COMMENT "",
  `d_lastdayinmonthfl` int(11) NOT NULL COMMENT "",
  `d_holidayfl` int(11) NOT NULL COMMENT "",
  `d_weekdayfl` int(11) NOT NULL COMMENT ""
)
DISTRIBUTED BY HASH(`d_datekey`) BUCKETS 1
PROPERTIES (
"replication_num" = "3"
);
```

## 数据准备

请参阅 **性能测试工具 -> [Star Schema Benchmark](/udoris/test/tool/ssb)**

## 数据导入

```shell
[root@xxxxx ssb-dbgen]# curl --location-trusted -u root:<创建集群时设置的管理密码>  -H "column_separator:|" -T supplier.tbl http://<任一Frontend节点地址>:8030/api/ssb/supplier/_stream_load
[root@xxxxx ssb-dbgen]# curl --location-trusted -u root:<创建集群时设置的管理密码>  -H "column_separator:|" -T customer.tbl http://<任一Frontend节点地址>:8030/api/ssb/customer/_stream_load
[root@xxxxx ssb-dbgen]# curl --location-trusted -u root:<创建集群时设置的管理密码>  -H "column_separator:|" -T date.tbl http://<任一Frontend节点地址>:8030/api/ssb/date/_stream_load
[root@xxxxx ssb-dbgen]# curl --location-trusted -u root:<创建集群时设置的管理密码>  -H "column_separator:|" -T lineorder.csv http://<任一Frontend节点地址>:8030/api/ssb/lineorder/_stream_load
[root@xxxxx ssb-dbgen]# curl --location-trusted -u root:<创建集群时设置的管理密码>  -H "column_separator:|" -T part.tbl http://<任一Frontend节点地址>:8030/api/ssb/part/_stream_load
```

## 运行多表性能测试语句

请参阅 **性能测试指南 -> [多表性能测试语句](/udoris/test/multiple_query)**

