MySQL前缀索引是一种优化技术，允许你只对字符串字段的前几个字符创建索引，而不是对整个字段值建索引。这在处理长字符串字段时特别有用。

## 什么是前缀索引

前缀索引只使用字段值的前N个字符来创建索引，而不是使用完整的字段值。例如，对于一个很长的URL字段，你可能只需要前20个字符就能有效区分大部分记录。

## 创建前缀索引的语法

```sql
-- 创建表时定义前缀索引
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255),
    INDEX idx_email_prefix (email(10))
);

-- 在现有表上添加前缀索引
ALTER TABLE users ADD INDEX idx_email_prefix (email(10));

-- 或使用CREATE INDEX语句
CREATE INDEX idx_email_prefix ON users (email(10));
```

## 前缀长度的选择

选择合适的前缀长度很重要，需要在索引大小和选择性之间找到平衡：

```sql
-- 分析不同前缀长度的选择性
SELECT 
    COUNT(DISTINCT LEFT(email, 5)) / COUNT(*) as prefix_5,
    COUNT(DISTINCT LEFT(email, 10)) / COUNT(*) as prefix_10,
    COUNT(DISTINCT LEFT(email, 15)) / COUNT(*) as prefix_15,
    COUNT(DISTINCT email) / COUNT(*) as full_column
FROM users;
```

## 优点

1. **减少存储空间** - 索引占用的磁盘空间更小
2. **提高缓存效率** - 更多的索引页可以放入内存
3. **加快索引维护** - 插入、更新、删除操作更快
4. **减少I/O操作** - 读取索引时需要的磁盘访问更少

## 缺点

1. **不支持ORDER BY优化** - MySQL无法使用前缀索引进行排序优化
2. **不支持覆盖索引** - 无法仅通过索引获取完整字段值
3. **选择性可能降低** - 如果前缀太短，可能导致更多重复值

## 使用场景

前缀索引特别适用于：

- **长字符串字段** - 如URL、长文本描述
- **有明显前缀模式** - 如手机号码、邮编
- **查询主要使用前缀匹配** - 如 `LIKE 'prefix%'` 查询

## 实际例子

```sql
-- 假设有一个存储文件路径的表
CREATE TABLE files (
    id INT PRIMARY KEY,
    file_path VARCHAR(500),
    created_at TIMESTAMP
);

-- 分析路径前缀的分布
SELECT 
    LEFT(file_path, 20) as prefix_20,
    COUNT(*) as count
FROM files 
GROUP BY LEFT(file_path, 20)
ORDER BY count DESC
LIMIT 10;

-- 创建前缀索引
CREATE INDEX idx_file_path_prefix ON files (file_path(50));

-- 这个查询可以有效使用前缀索引
SELECT * FROM files WHERE file_path LIKE '/uploads/images/2024/%';

-- 但这个查询无法有效使用前缀索引
SELECT * FROM files ORDER BY file_path;
```

## 最佳实践

1. **测试不同前缀长度** - 找到选择性和存储空间的最佳平衡点
2. **考虑查询模式** - 确保你的查询能够有效利用前缀索引
3. **监控性能** - 定期检查前缀索引的效果
4. **避免过短的前缀** - 太短的前缀可能导致索引效果不佳

前缀索引是MySQL中一个强大的优化工具，特别适用于处理长字符串字段的场景。正确使用可以显著提高查询性能并减少存储开销。