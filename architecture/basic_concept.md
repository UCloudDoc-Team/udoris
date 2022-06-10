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

## Tablet & Partition

在 Doris 的存储引擎中，用户数据被水平划分为若干个数据分片（Tablet，也称作数据分桶）。每个 Tablet 包含若干数据行。各个 Tablet 之间的数据没有交集，并且在物理上是独立存储的。多个 Tablet 在逻辑上归属于不同的分区（Partition）。一个 Tablet 只属于一个 Partition。而一个 Partition 包含若干个 Tablet。若干个 Partition 组成一个 Table。Partition 可以视为是逻辑上最小的管理单元。数据的导入与删除，都可以或仅能针对一个 Partition 进行

### Row & Column

Row：即用户的一行数据，Column： 用于描述一行数据中不同的字段，Column 可以分为两大类：Key 和 Value。从业务角度看，Key 和 Value 可以分别对应维度列和指标列。从聚合模型的角度来说，Key 列相同的行，会聚合成一行。其中 Value 列的聚合方式由用户在建表时指定

### 

