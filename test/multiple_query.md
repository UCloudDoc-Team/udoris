# 多表性能测试语句

## 基于Star Schema Benchmark工具

- #### Q1.1

  ```sql
  SELECT SUM(lo_extendedprice*lo_discount) AS REVENUE FROM lineorder, date 
  WHERE lo_orderdate = d_datekey AND d_year = 1993 
  AND lo_discount BETWEEN 1 AND 3 
  AND lo_quantity < 25;
  ```

- #### Q1.2

  ```sql
  SELECT SUM(lo_extendedprice*lo_discount) AS REVENUE FROM  lineorder, date 
  WHERE lo_orderdate = d_datekey 
  AND d_yearmonth = 'Jan1994' 
  AND lo_discount BETWEEN 4 AND 6 
  AND lo_quantity BETWEEN 26 AND 35;
  ```

- #### Q1.3

  ```sql
  SELECT SUM(lo_extendedprice*lo_discount) AS REVENUE FROM  lineorder, date 
  WHERE lo_orderdate = d_datekey 
  AND d_weeknuminyear= 6
  AND d_year = 1994 
  AND lo_discount BETWEEN 5 AND 7 
  AND lo_quantity BETWEEN 26 AND 35;
  ```

- #### Q2.1

  ```sql
  SELECT SUM(lo_revenue), d_year, p_brand FROM lineorder, date, part, supplier
  WHERE lo_orderdate = d_datekey 
  AND lo_partkey = p_partkey
  AND lo_suppkey = s_suppkey 
  AND p_category = 'MFGR#12' 
  AND s_region = 'AMERICA' 
  GROUP BY d_year, p_brand 
  ORDER BY d_year, p_brand;
  ```

- #### Q2.2

  ```sql
  SELECT 
  SUM(lo_revenue), d_year, p_brand FROM lineorder, date, part, supplier 
  WHERE lo_orderdate = d_datekey 
  AND lo_partkey = p_partkey
  AND lo_suppkey = s_suppkey 
  AND p_brand BETWEEN 'MFGR#2221' AND 'MFGR#2228' 
  AND s_region = 'ASIA' 
  GROUP BY d_year, p_brand 
  ORDER BY d_year, p_brand;
  ```

- #### Q2.3

  ```sql
  SELECT SUM(lo_revenue), d_year, p_brand FROM lineorder, date, part, supplier 
  WHERE lo_orderdate = d_datekey 
  AND lo_partkey = p_partkey
  AND lo_suppkey = s_suppkey 
  AND p_brand = 'MFGR#2239' 
  AND s_region = 'EUROPE' 
  GROUP BY d_year, p_brand 
  ORDER BY d_year, p_brand;
  ```

- #### Q3.1

  ```sql
  SELECT c_nation, s_nation, d_year, SUM(lo_revenue) AS REVENUE FROM customer, lineorder, supplier, date 
  WHERE lo_custkey = c_custkey
  AND lo_suppkey = s_suppkey 
  AND lo_orderdate = d_datekey 
  AND c_region = 'ASIA' 
  AND s_region = 'ASIA' 
  AND d_year >= 1992 
  AND d_year <= 1997
  GROUP BY c_nation, s_nation, d_year 
  ORDER BY d_year ASC, REVENUE DESC;
  ```

- #### Q3.2

  ```sql
  SELECT c_city, s_city, d_year, sum(lo_revenue) AS REVENUE FROM customer, lineorder, supplier, date 
  WHERE lo_custkey = c_custkey
  AND lo_suppkey = s_suppkey 
  AND lo_orderdate = d_datekey 
  AND c_nation = 'UNITED STATES' 
  AND s_nation = 'UNITED STATES' 
  AND d_year >= 1992 
  AND d_year <= 1997 
  GROUP BY c_city, s_city, d_year 
  ORDER BY d_year ASC, REVENUE DESC;
  ```

- #### Q3.3

  ```sql
  SELECT c_city, s_city, d_year, SUM(lo_revenue) AS  REVENUE FROM customer, lineorder, supplier, date 
  WHERE lo_custkey = c_custkey
  AND lo_suppkey = s_suppkey 
  AND lo_orderdate = d_datekey 
  AND (c_city='UNITED KI1' OR c_city='UNITED KI5') 
  AND (s_city='UNITED KI1' OR s_city='UNITED KI5') 
  AND d_year >= 1992 
  AND d_year <= 1997
  GROUP BY c_city, s_city, d_year
  ORDER BY d_year ASC, REVENUE DESC;
  ```

- #### Q3.4

  ```sql
  SELECT c_city, s_city, d_year, SUM(lo_revenue) AS REVENUE FROM customer, lineorder, supplier, date 
  WHERE lo_custkey = c_custkey 
  AND lo_suppkey = s_suppkey 
  AND lo_orderdate = d_datekey 
  AND (c_city='UNITED KI1' OR c_city='UNITED KI5') 
  AND (s_city='UNITED KI1' OR s_city='UNITED KI5') 
  AND d_yearmonth = 'Dec1997'
  GROUP BY c_city, s_city, d_year
  ORDER BY d_year ASC,  REVENUE DESC;
  ```

- #### Q4.1

  ```sql
  SELECT d_year, c_nation,SUM(lo_revenue - lo_supplycost) AS PROFIT FROM date, customer, supplier, part, lineorder
  WHERE lo_custkey = c_custkey 
  AND lo_suppkey = s_suppkey 
  AND lo_partkey = p_partkey 
  AND lo_orderdate = d_datekey
  AND c_region = 'AMERICA' 
  AND s_region = 'AMERICA' 
  AND (p_mfgr = 'MFGR#1' OR p_mfgr = 'MFGR#2')
  GROUP BY d_year, c_nation
  ORDER BY d_year, c_nation;
  ```

- #### Q4.2

  ```sql
  SELECT d_year, s_nation, p_category, SUM(lo_revenue - lo_supplycost) AS PROFIT FROM date, customer, supplier, part, lineorder
  WHERE lo_custkey = c_custkey 
  AND lo_suppkey = s_suppkey 
  AND lo_partkey = p_partkey 
  AND lo_orderdate = d_datekey 
  AND c_region = 'AMERICA'
  AND s_region = 'AMERICA' 
  AND (d_year = 1997 OR d_year = 1998) 
  AND (p_mfgr = 'MFGR#1' OR p_mfgr = 'MFGR#2')
  GROUP BY d_year, s_nation, p_category
  ORDER BY d_year, s_nation, p_category;
  ```

- #### Q4.3

  ```sql
  SELECT d_year, s_city, p_brand, SUM(lo_revenue - lo_supplycost) AS PROFIT FROM date, customer, supplier, part, lineorder
  WHERE lo_custkey = c_custkey 
  AND lo_suppkey = s_suppkey
  AND lo_partkey = p_partkey
  AND lo_orderdate = d_datekey
  AND s_nation = 'UNITED STATES'
  AND (d_year = 1997 OR d_year = 1998)
  AND p_category = 'MFGR#14'
  GROUP BY d_year, s_city, p_brand
  ORDER BY d_year, s_city, p_brand;
  ```
