# mysql除了唯一索引还有其他的索引吗

MySQL确实有多种索引类型，除了唯一索引外，还有以下几种：

## 主要索引类型

**普通索引（Normal Index）**

- 最基本的索引类型，没有任何限制
- 用于加速查询，允许重复值

```sql
CREATE INDEX idx_name ON table_name(column_name);
```

**主键索引（Primary Key Index）**

- 特殊的唯一索引，不允许NULL值
- 一个表只能有一个主键索引
- InnoDB引擎中，主键索引是聚簇索引

**复合索引（Composite Index）**

- 在多个列上创建的索引
- 遵循最左前缀匹配原则

```sql
CREATE INDEX idx_name_age ON users(name, age);
```

**全文索引（Full-Text Index）**

- 用于文本搜索，支持自然语言搜索
- 主要用于CHAR、VARCHAR、TEXT类型

```sql
CREATE FULLTEXT INDEX idx_content ON articles(content);
```

## 按数据结构分类

**B+Tree索引**

- MySQL默认索引类型
- 适合范围查询和排序

**Hash索引**

- Memory存储引擎支持
- 等值查询速度极快，不支持范围查询

**R-Tree索引**

- 用于地理空间数据类型
- 支持空间数据的快速检索

## 按物理存储分类

**聚簇索引（Clustered Index）**

- 数据行按索引顺序存储
- InnoDB中主键就是聚簇索引

**非聚簇索引（Non-Clustered Index）**

- 索引和数据分别存储
- 二级索引都是非聚簇索引

## 扩展知识

**索引优化建议：**

1. 为经常查询的列创建索引
2. 避免在低选择性列上创建索引
3. 合理使用复合索引
4. 定期分析和优化索引使用情况

**性能影响：**

- 索引加速查询但会降低写入性能
- 索引需要额外的存储空间
- 过多索引会影响优化器选择

选择合适的索引类型需要根据具体的查询模式和数据特征来决定。