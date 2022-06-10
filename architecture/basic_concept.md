# 基本概念

## 地域（Region）

购买云数据仓库 UDoris的服务器所处地理位置，您可按需要选择云数据仓库 UDoris集群所在的Region。

### 云数据仓库 UDoris集群

云数据仓库 UDoris是由一个或三个Frontend节点和三个以上的Backend节点组成的集群。

## 副本（Replica）

数据和元数据都默认使用3副本存储，可根据创建的Backend节点数指定相应的副本数。

## Frontend

Doris的前端节点，简称FE，以java语言为主，主要负责接受和返回客户端请求，元数据以及集群管理，查询计划生成等工作。
FE 分为 Leader，Follower 和 Observer 三种角色。3台FE中包含1个 Leader角色，2个 Follower角色时FE处于高可用

## Backend

Doris的后端节点，简称BE，以C++语言为主，主要负责数据存储与管理、查询计划执行等工作。

### Tablet

Doris 中表的逻辑分片，也是 Doris 中副本管理的基本单位，每个表根据分区和分桶机制被划分成多个 Tablet 存储在不同 BE 节点上。
