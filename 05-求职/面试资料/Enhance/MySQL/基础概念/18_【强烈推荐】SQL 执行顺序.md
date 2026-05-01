(1) FROM  
(2) JOIN  
(3) ON  
(4) WHERE  
(5) GROUP BY  
(6) WITH  
(7) HAVING  
(8) SELECT  
(9) DISTINCT  
(10) ORDER BY  
(11) LIMIT

**1. FROM** - 确定数据源
- 首先处理 FROM 子句，确定要查询的表
- 如果有 JOIN，会在这个阶段处理表之间的连接
**2. WHERE** - 行级过滤
- 对 FROM 阶段产生的数据进行行级过滤
- 只有满足 WHERE 条件的行才会进入下一阶段
**3. GROUP BY** - 分组
- 将满足 WHERE 条件的行按照指定列进行分组
**4. HAVING** - 组级过滤
- 对分组后的结果进行过滤
- HAVING 是在 GROUP BY 之后执行的，所以可以使用聚合函数
**5. SELECT** - 选择列
- 选择要返回的列
- 计算表达式和聚合函数
**6. DISTINCT** - 去重
- 如果使用了 DISTINCT，在这个阶段去除重复行
**7. ORDER BY** - 排序
- 对最终结果进行排序
**8. LIMIT/OFFSET** - 限制结果集
- 限制返回的行数和起始位置