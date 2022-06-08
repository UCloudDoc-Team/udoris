# 500亿数据性能测试

## 生成测试数据

请参阅性能测试工具[Star Schema Benchmark](/udoris/test/tool/ssb)

**生成数据如下：**

| 表名      | 行数                 | 大小 | 描述       |
| --------- | -------------------- | ---- | ---------- |
| lineorder | 509亿（50999977875） | 5.2T | 商品订单表 |
| customer  | 2.55亿（255000000）  | 24G  | 客户表     |
| part      | 280万（2800000）     | 232M | 零部件表   |
| supplier  | 1700万（17000000）   | 1.4G | 供应商表   |
| date      | 2556                 | 225K | 日期表     |
| 合计      | 51274780431          | 5.44 |            |

## 云数据仓库UDoris性能测试

### **1.0.0版本,Frontend/4核/16G/RSSD云盘/1节点,Backend/64核/512G/RSSD云盘/6节点**

以下数据结果基于Star Schema Benchmark工具进行的**多表测试**。

| 测试语句 | 耗时（ms） |
| -------- | ---------- |
| Q1.1     | 13442      |
| Q1.2     | 7974       |
| Q1.3     | 11590      |
| Q2.1     | 90000      |
| Q2.2     | 109000     |
| Q2.3     | 61000      |
| Q3.1     | 3323000    |
| Q3.2     | 72000      |
| Q3.3     | 32400      |
| Q3.4     | 67000      |
| Q4.1     | 231000     |
| Q4.2     | 157000     |
| Q4.3     | 64790      |
