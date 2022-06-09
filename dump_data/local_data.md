# 本地数据导入

## 注意事项

- Stream Load 用于将本地文件导入到 Doris 中。
- 建议一个导入请求的数据量控制在 1 - 2 GB 以内。如果有大量本地文件，可以分批并发提交

## 操作步骤

### 准备测试数据

创建csv文件lineorder.csv放到之前创建的云主机数据盘/data目录下

```
1|2|28720|133109|1289|19960502|3-MEDIUM|0|44|5025240|11485668|8|4623220|68526|0|19960722|RAIL
2|3|28720|152761|373|19960502|3-MEDIUM|0|3|544128|11485668|4|522362|108825|1|19960602|MAIL
3|1|6415|176345|1060|19970713|4-NOT SPECI|0|50|7106700|6890656|4|6822432|85280|1|19970907|FOB
4|1|11090|25360|43|19930728|3-MEDIUM|0|24|3084864|9275090|2|3023166|77121|3|19931007|MAIL
5|2|11090|10463|1321|19930728|3-MEDIUM|0|46|6317916|9275090|8|5812482|82407|6|19930921|RAIL
6|1|22681|7272|1235|19930725|1-URGENT|0|32|3773664|6321665|7|3509507|70756|1|19930930|REG AIR
7|2|22681|6452|1387|19930725|1-URGENT|0|7|950915|6321665|4|912878|81507|0|19930923|SHIP
8|3|22681|37131|1202|19930725|1-URGENT|0|18|1922634|6321665|4|1845728|64087|1|19930828|FOB
9|1|22013|32255|305|19960831|2-HIGH|0|5|593625|3762529|4|569880|71235|3|19961119|MAIL
10|2|22013|96127|478|19960831|2-HIGH|0|28|3144736|3762529|1|3113288|67387|2|19961001|AIR
```

### 创建数据库及数据表

- 通过mysql-client连接集群

  ```shell
  mysql -uroot -h<任一frontend节点IP地址> -P9030 -p<创建集群时设置的密码>
  ```


- 建库建表

```sql
CREATE DATABASE IF NOT EXISTS ssb;
CREATE TABLE `lineorder` (
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
```

- 使用以下命令在之前创建的云主机导入本地数据

```shell
curl --location-trusted -u root:<创建集群时设置的密码> -H "column_separator:|" -T /data/lineorder.csv http://<任一Frontend节点IP地址>:8030/api/ssb/lineorder/_stream_load
```

- Java 代码示例（这里通过一个简单的 JAVA 示例来执行 Stream Load）

```java
package demo.doris;

import org.apache.commons.codec.binary.Base64;
import org.apache.http.HttpHeaders;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPut;
import org.apache.http.entity.FileEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.DefaultRedirectStrategy;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.File;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/*
这是一个 Doris Stream Load 示例，需要依赖
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>
 */
public class DorisStreamLoader {
    //可以选择填写 frontend 地址以及 frontend 的 http_port，但须保证客户端和 BE 节点的连通性。
    private final static String HOST = "your_host";//集群详情给出的连接地址，也就是Frontend的master角色对应的ip
    private final static int PORT = 8030;
    private final static String DATABASE = "ssb";   // 要导入的数据库
    private final static String TABLE = "lineorder";     // 要导入的表
    private final static String USER = "root";   
    private final static String PASSWD = "";        // 创建集群时的密码
    private final static String LOAD_FILE_NAME = "/path/to/1.txt"; // 要导入的本地文件路径

    private final static String loadUrl = String.format("http://%s:%s/api/%s/%s/_stream_load",
            HOST, PORT, DATABASE, TABLE);

    private final static HttpClientBuilder httpClientBuilder = HttpClients
            .custom()
            .setRedirectStrategy(new DefaultRedirectStrategy() {
                @Override
                protected boolean isRedirectable(String method) {
                    // 如果连接目标是 frontend，则需要处理 307 redirect。
                    return true;
                }
            });

    public void load(File file) throws Exception {
        try (CloseableHttpClient client = httpClientBuilder.build()) {
            HttpPut put = new HttpPut(loadUrl);
            put.setHeader(HttpHeaders.EXPECT, "100-continue");
            put.setHeader(HttpHeaders.AUTHORIZATION, basicAuthHeader(USER, PASSWD));

            // 可以在 Header 中设置 stream load 相关属性，这里我们设置 label 和 column_separator。
            put.setHeader("label","label1");
            put.setHeader("column_separator",",");

            // 设置导入文件。
            // 这里也可以使用 StringEntity 来传输任意数据。
            FileEntity entity = new FileEntity(file);
            put.setEntity(entity);

            try (CloseableHttpResponse response = client.execute(put)) {
                String loadResult = "";
                if (response.getEntity() != null) {
                    loadResult = EntityUtils.toString(response.getEntity());
                }

                final int statusCode = response.getStatusLine().getStatusCode();
                if (statusCode != 200) {
                    throw new IOException(
                            String.format("Stream load failed. status: %s load result: %s", statusCode, loadResult));
                }

                System.out.println("Get load result: " + loadResult);
            }
        }
    }

    private String basicAuthHeader(String username, String password) {
        final String tobeEncode = username + ":" + password;
        byte[] encoded = Base64.encodeBase64(tobeEncode.getBytes(StandardCharsets.UTF_8));
        return "Basic " + new String(encoded);
    }

    public static void main(String[] args) throws Exception{
        DorisStreamLoader loader = new DorisStreamLoader();
        File file = new File(LOAD_FILE_NAME);
        loader.load(file);
    }
}
```

- 使用mysql-client非交互模式查询验证

```shell
mysql -uroot -h<任一Frontend节点IP地址> -P9030 -p<创建集群时设置的密码> -e "select * from ssb.lineorder";
```

