# 聚合函数

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>描述</th>
        <th>语法</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>sum</td>
        <td>统计行数或者非 NULL 值个数</td>
        <td>sum(expr)</td>
    </tr>
    <tr>
        <td>min</a></td>
        <td>求最小值</td>
        <td>min(column)</td>
    </tr>
		<tr>
        <td>max</a></td>
        <td>求最大值</td>
        <td>max(column)</td>
    </tr>
    <tr>
        <td>count</td>
        <td>统计行数或者非 NULL 值个数</td>
        <td>count(expr)、COUNT(DISTINCT expr)、count()、count(*)</td>
    </tr>
    <tr>
        <td>bitmap_union</td>
        <td>去重</td>
        <td>BITMAP_UNION(expr) : 计算输入 Bitmap 的并集，返回新的bitmap
						BITMAP_UNION_COUNT(expr): 计算输入 Bitmap 的并集，返回其基数
						BITMAP_UNION_INT(expr)：计算 TINYINT,SMALLINT 和 INT 类型的列中不同值的个数，返回值和COUNT(DISTINCT expr) 相同</td>
    </tr>
    <tr>
        <td>hll_union</td>
        <td>非精确快速去重</td>
        <td>HLL_UNION(HLL_HASH(COLUMN)) column 列的类型不能是 DECIMAL , 或者 HLL_UNION(COLUMN) 且 base 表为 AGG 模型</td>
    </tr>
    </tbody>
</table>

