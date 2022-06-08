# 建库（Create Database） 

**基本语法：**

```sql
CREATE DATABASE [IF NOT EXISTS] db_name;
```

当数据库已经存在时，避免执行报错，可指定 IF NOT EXISTS 关键字，指定后重复执行语句不会报错且也不会重建数据库。

**示例：**

```sql
CREATE DATABASE IF NOT EXISTS ssb;
```

