# 使用JDBC同步数据

用户可以通过 JDBC 协议，使用 INSERT 语句进行数据导入。

## 操作步骤

- 通过mysql-client连接集群

```shell
mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
```

- 建库建表

```sql
CREATE DATABASE IF NOT EXISTS ssb;
CREATE TABLE IF NOT EXISTS example_tbl
(
    id INT,
    age TINYINT,
    name VARCHAR(50)
)
unique key(id)
DISTRIBUTED BY HASH(id) BUCKETS 3;
```

- 写入数据

```sql
INSERT INTO example_tbl VALUES
(1, 18, "张三")
(2, 19, "李四")
(3, 20, "王五");
```

- JDBC示例（这里我们给出一个简单的 JDBC 批量 INSERT 代码示例）

```java
package demo.doris;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class DorisJdbcLoader {
    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    private static final String DB_URL_PATTERN = "jdbc:mysql://%s:%d/%s?rewriteBatchedStatements=true";
    private static final String HOST = "your_host"; //集群详情给出的连接地址，也就是Frontend的master角色对应的ip
    private static final int PORT = 8030;
    private static final String DB = "ssb";
    private static final String TBL = "example_tbl";
    private static final String USER = "root";
    private static final String PASSWD = "my_pass";//创建集群时设置的密码
    private static final int INSERT_BATCH_SIZE = 10000;

    public static void main(String[] args) throws Exception {
        insert();
    }

    private static void insert() throws Exception {
        String query = "insert into " + TBL + " values(?, ?)";
        String dbUrl = String.format(DB_URL_PATTERN, HOST, PORT, DB);
        Class.forName(JDBC_DRIVER);
        try (Connection conn = DriverManager.getConnection(dbUrl, USER, PASSWD)) {
            PreparedStatement stmt = conn.prepareStatement(query);
            for (int i =0; i < INSERT_BATCH_SIZE; i++) {
                stmt.setInt(1, i);
                stmt.setInt(2, i * 100);
                stmt.addBatch();
            }
            int[] res = stmt.executeBatch();
            System.out.println(res);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 使用mysql-client非交互模式查询验证

```shell
mysql -uroot -h<任一Frontend节点IP地址> -P9030 -p<创建集群时设置的密码> -e "select * from ssb.example_tbl";
```

