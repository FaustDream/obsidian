MySQL 中 UNION 和 UNION ALL 的主要区别在于对重复记录的处理方式：

## UNION

- **去重**：会自动去除结果集中的重复记录
- **性能较慢**：因为需要进行去重操作，会消耗更多资源
- **排序**：默认会对结果进行排序（为了去重）

## UNION ALL

- **保留重复**：保留所有记录，包括重复的记录
- **性能较快**：不需要去重和排序操作，执行效率更高
- **无排序**：按照各个查询的顺序直接合并结果

## 示例对比

假设有两个表的数据：

```sql
-- 表1数据
SELECT id, name FROM table1;
-- 结果：1, 'Alice'  2, 'Bob'

-- 表2数据  
SELECT id, name FROM table2;
-- 结果：2, 'Bob'  3, 'Charlie'
```

使用 UNION：

```sql
SELECT id, name FROM table1
UNION
SELECT id, name FROM table2;
-- 结果：1, 'Alice'  2, 'Bob'  3, 'Charlie' (去除了重复的 Bob)
```

使用 UNION ALL：

```sql
SELECT id, name FROM table1
UNION ALL
SELECT id, name FROM table2;
-- 结果：1, 'Alice'  2, 'Bob'  2, 'Bob'  3, 'Charlie' (保留重复的 Bob)
```

## 使用建议

- 如果确定没有重复数据或需要保留重复数据，优先使用 **UNION ALL**，性能更好
- 只有在明确需要去重时才使用 **UNION**
- 两个查询的列数和对应列的数据类型必须一致