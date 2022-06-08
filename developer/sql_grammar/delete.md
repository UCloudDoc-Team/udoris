# 删除数据

该语句用于按条件删除指定 table（base index） partition 中的数据。

该操作会同时删除和此 base index 相关的 rollup index 的数据。

**基本语法：**

```
DELETE FROM table_name [PARTITION partition_name | PARTITIONS (p1, p2)]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...];
```

### 示例

1. 删除 my_table partition p1 中 k1 列值为 3 的数据行

   ```sql
   DELETE FROM my_table PARTITION p1
       WHERE k1 = 3;
   ```

2. 删除 my_table partition p1 中 k1 列值大于等于 3 且 k2 列值为 "abc" 的数据行

   ```sql
   DELETE FROM my_table PARTITION p1
   WHERE k1 >= 3 AND k2 = "abc";
   ```

3. 删除 my_table partition p1, p2 中 k1 列值大于等于 3 且 k2 列值为 "abc" 的数据行

   ```sql
   DELETE FROM my_table PARTITIONS (p1, p2)
   WHERE k1 >= 3 AND k2 = "abc";
   ```
