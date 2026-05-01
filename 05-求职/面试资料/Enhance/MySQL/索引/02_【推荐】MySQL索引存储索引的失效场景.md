# MySQL索引存储？索引的失效场景？

## MySQL索引存储机制

### 1. 索引的物理存储结构

**B+Tree存储结构**

- MySQL的InnoDB引擎使用B+Tree作为索引的存储结构
- 非叶子节点只存储键值，不存储数据
- 所有数据都存储在叶子节点中，叶子节点之间通过指针连接形成链表
- 这种结构使得范围查询和顺序扫描效率很高

**聚簇索引与非聚簇索引**

- **聚簇索引**：数据和索引存储在一起，InnoDB表的主键就是聚簇索引
- **非聚簇索引**：索引和数据分开存储，叶子节点存储的是指向数据行的指针

### 2. 索引文件存储

**InnoDB存储引擎**

- 索引和数据存储在同一个文件中（.ibd文件）
- 主键索引的叶子节点直接存储行数据
- 辅助索引的叶子节点存储主键值，需要回表查询

**MyISAM存储引擎**

- 索引文件（.MYI）和数据文件（.MYD）分离
- 所有索引都是非聚簇索引
- 索引文件中存储的是数据记录的物理地址

## 索引失效场景详解

### 1. 查询条件导致的失效

**使用函数或表达式**

```sql
-- 失效示例
SELECT * FROM users WHERE YEAR(create_time) = 2024;
SELECT * FROM users WHERE age + 1 = 25;

-- 正确写法
SELECT * FROM users WHERE create_time >= '2024-01-01' AND create_time < '2025-01-01';
SELECT * FROM users WHERE age = 24;
```

**类型转换**

```sql
-- 失效：字符串字段与数字比较
SELECT * FROM users WHERE phone = 13800138000;

-- 正确：保持类型一致
SELECT * FROM users WHERE phone = '13800138000';
```

### 2. LIKE查询的失效情况

**前缀通配符**

```sql
-- 失效：以%开头
SELECT * FROM users WHERE name LIKE '%张%';

-- 有效：以具体字符开头
SELECT * FROM users WHERE name LIKE '张%';
```

### 3. 复合索引的失效

**违反最左前缀原则**

```sql
-- 复合索引：(name, age, city)
-- 失效：跳过了name
SELECT * FROM users WHERE age = 25 AND city = '北京';

-- 有效：遵循最左前缀
SELECT * FROM users WHERE name = '张三' AND age = 25;
```

### 4. OR条件导致的失效
> 当使用 `OR` 连接不同列的条件时，无法同时利用联合索引来满足两个独立的条件

```sql
-- 失效：OR连接的条件中有未建索引的字段
-- 要么匹配 `id = 1`，要么匹配 `email = 'test@example.com'`，但不会同时匹配 id 和 email
SELECT * FROM users WHERE id = 1 OR email = 'test@example.com';

-- 解决方案：使用UNION或为所有字段建立索引
SELECT * FROM users WHERE id = 1 
UNION 
SELECT * FROM users WHERE email = 'test@example.com';
```

### 5. 范围查询后的条件失效

```sql
-- 复合索引：(status, create_time, name)
-- name字段的索引会失效
SELECT * FROM users WHERE status = 1 AND create_time > '2024-01-01' AND name = '张三';
```

### 6. 数据分布导致的失效

**数据重复度过高**

```sql
-- 如果gender字段的选择性很低（只有男/女），优化器可能选择全表扫描
SELECT * FROM users WHERE gender = '男';
```

**查询数据量过大**

```sql
-- 查询结果超过表的30%时，优化器倾向于全表扫描
SELECT * FROM users WHERE age > 18; -- 如果大部分用户都成年
```

## 扩展知识点

### 1. 索引优化策略

**覆盖索引**

```sql
-- 创建覆盖索引，避免回表
CREATE INDEX idx_user_info ON users(name, age, email);
SELECT name, age, email FROM users WHERE name = '张三';
```

**索引下推**

```sql
-- MySQL 5.6+支持索引下推优化
-- 在索引遍历过程中，先过滤条件，减少回表次数
SELECT * FROM users WHERE name LIKE '张%' AND age > 20;
```

### 2. 监控索引使用情况

```sql
-- 查看索引使用统计
SHOW INDEX FROM users;

-- 检查未使用的索引
SELECT * FROM sys.schema_unused_indexes;

-- 分析查询执行计划
EXPLAIN SELECT * FROM users WHERE name = '张三';
```

### 3. 索引维护建议

**定期重建索引**

```sql
-- 重建索引以消除碎片
ALTER TABLE users ENGINE=InnoDB;
OPTIMIZE TABLE users;
```

**监控索引大小**

```sql
-- 查看索引大小
SELECT 
    table_name,
    index_name,
    ROUND(stat_value * @@innodb_page_size / 1024 / 1024, 2) AS 'Size(MB)'
FROM mysql.innodb_index_stats 
WHERE stat_name = 'size' AND table_name = 'users';
```

理解这些索引存储机制和失效场景，能帮助我们更好地设计数据库结构，编写高效的SQL查询，并在性能调优时快速定位问题。