# 产品购买与使用

## 为什么只提供一种云盘类型？

Doris的存储特性对磁盘吞吐量要求很高，为保证Doris的性能优势， 因此仅提供RSSD云盘。

## 配置升降级对集群有什么影响？

当进行配置升降级时，集群将处于升级中状态并持续数分钟，期间集群将不能对外服务，请您合理选择升级时间。

## 配置升级有什么建议？

当数据量增长到一定级别时，可能当前集群配置满足不了您的需求，您可以按照业务增长情况选择升级配置。

## 如何连接云数据仓库UDoris？

为保证安全，云数据仓库 UDW Apache Doris仅提供内网VPC网络，您连接集群时可以配合同一地域的云主机或者网络产品使用。