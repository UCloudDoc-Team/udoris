# Doris 常见问题

## 怎么通过 SQL 对审计日志进行查看和分析

创建集群时该插件已经自动安装，查询和使用审计日志的步骤如下

- 通过mysql-client连接集群

  ```shell
  mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
  ```

- show databases;

  ```shell
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |               |
  | udoris             |
  +--------------------+
  2 rows in set (0.00 sec)
  ```

- use udoris;show tables;

  ```shell
  mysql> use udoris;show tables;
  Database changed
  +------------------+
  | Tables_in_udoris |
  +------------------+
  | audit_log        |
  +------------------+
  1 row in set (0.00 sec)
  
  ```

审计日志位于udoris.audit_log表中。插件会不断的将审计日志插入到这个表中。

### 查询报错：Failed to get scan range, no queryable replica found in tablet: xxxx

这种情况是因为对应的 tablet 没有找到可以查询的副本，通常原因可能是 BE 宕机、副本缺失等。可以先通过 `show proc '/backends'` 语句，查询BE是否全部存活，如果宕机，则需要重启。

同时可以通过 `show tablet tablet_id` 语句，然后执行后面的 `show proc` 语句，查看这个 tablet 对应的副本信息，检查副本是否完整。同时还可以通过 `show proc "/cluster_balance"` 信息来查询集群内副本调度和修复的进度。

### 创建表报错：Failed to find enough host with storage medium and tag(HDD/{"location" : "default"}) in all backends. need: xxxx

这是因为创建表指定的副本数大于了Backend节点的数量，副本数默认是3，如果手动指定，需要小于等于Backend节点的数量。

### 查询超时：detailMessage = query timeout

udoris支持最长查询时间为6000s, 如果仍然未完成, 会被cancel掉

### 查询报内存不够：detailMessage = Memory limit exceeded

udoris对每个用户设置内存限制为Backend节点内存*70%

### Doris 是否支持修改列名？

不支持修改列名。

Doris支持修改数据库名、表名、分区名、物化视图（Rollup）名称，以及列的类型、注释、默认值等等。但遗憾的是，目前不支持修改列名。

### 是否支持Broker Load方式导入数据？

udoris默认不开启broker,如需要使用，[请联系技术支持](https://spt.ucloud.cn/)

更多常见问题请参考[官网常见问题](https://doris.apache.org/docs/faq/data-faq)

