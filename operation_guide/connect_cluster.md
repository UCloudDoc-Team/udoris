# 连接集群

## Mysql命令行工具连接

### 操作步骤

  1. 登录UCloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Doris”，进入到[数据仓库 UDW Apache Doris控制台](https://console.ucloud.cn/udw/doris)

  2. 在**集群列表**页面，点击**详情**可查看集群节点列表，列表中已列出节点地址。

  3. 下载mysql-client，官方下载地址：[下载mysql-client](https://dev.mysql.com/downloads/mysql/)。安装请参见[快速上手](/udoris/gettingstart)

  4. 连接云数据仓库 UDW Apache Doris，在同一地域下的云主机上执行以下命令。

     ```
     mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
     ```

参数说明如下

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| -h   | 任一Frontend节点的ip地址(建议用详情页给出的CLI连接地址)。<br />要求mysql-client所在云主机和云数据仓库 UDW Apache Doris在同一地域内（同一网段） |
| -p   | Doris Frontend 通过 mysql 协议查询连接端口，默认9030         |
| -u   | 管理用户名，默认root                                         |
| -p   | 您创建集群时设置的管理密码                                   |

## JDBC连接数据库

### 操作步骤

1. 使用maven管理项目下载依赖包：

```xml
<dependencies>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.11</version>
    </dependency>
</dependencies>
```

2. 代码示例：

```java
import java.sql.*;

public class DorisConnect {
    public static void main(String[] args) throws Exception {
        Class.forName("com.mysql.jdbc.Driver");//加载驱动
        String url = "jdbc:mysql://node_ip:9030/ssb"; //node_ip为frontend节点的ip
        try(Connection conn = DriverManager.getConnection(url, "root", password)){//password是登陆集群的密码
            PreparedStatement stmt = conn.prepareStatement("select * from ssb.lineorder");
            ResultSet resultSet = stmt.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            while (resultSet.next()){
                for (int i = 1; i <= columnCount; ++i) {
                    System.out.println(metaData.getColumnName(i) + ":" + resultSet.getString(i));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

