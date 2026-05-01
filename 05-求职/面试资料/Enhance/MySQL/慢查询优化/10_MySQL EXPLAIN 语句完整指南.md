# MySQL EXPLAIN 语句完整指南

## 基本语法

```sql
EXPLAIN [EXTENDED | PARTITIONS | FORMAT=JSON] 
SELECT * FROM table_name WHERE condition;
```

## EXPLAIN 输出字段详解

### 1. id - 查询标识符

表示查询中执行 SELECT 的顺序和层次。

```sql
-- 示例表结构
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    city_id INT
);

CREATE TABLE cities (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 简单查询 (id = 1)
EXPLAIN SELECT * FROM users WHERE age > 25;

-- 子查询 (id = 1, 2)
EXPLAIN SELECT * FROM users 
WHERE city_id IN (SELECT id FROM cities WHERE name = 'Beijing');

-- 联合查询 (id = 1, 2)
EXPLAIN 
SELECT name FROM users WHERE age > 30
UNION 
SELECT name FROM users WHERE age < 20;
```

### 2. select_type - 查询类型

#### SIMPLE - 简单查询

```sql
EXPLAIN SELECT * FROM users WHERE name = 'John';
```

#### PRIMARY - 主查询

```sql
EXPLAIN SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders);
```

#### SUBQUERY - 子查询

```sql
EXPLAIN SELECT *, 
    (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;
```

#### DERIVED - 派生表

```sql
EXPLAIN SELECT * FROM 
(SELECT city_id, COUNT(*) as user_count FROM users GROUP BY city_id) as city_stats
WHERE user_count > 10;
```

#### UNION - 联合查询

```sql
EXPLAIN 
SELECT name FROM users WHERE age > 30
UNION 
SELECT name FROM users WHERE city_id = 1;
```

### 3. table - 表名

显示当前行涉及的表名或别名。

```sql
EXPLAIN SELECT u.name, c.name 
FROM users u 
JOIN cities c ON u.city_id = c.id;
-- table 字段会分别显示 u 和 c
```

### 4. partitions - 分区信息

对于分区表，显示查询涉及的分区。

```sql
-- 创建分区表示例
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023)
);

EXPLAIN PARTITIONS 
SELECT * FROM sales WHERE sale_date = '2021-06-15';
-- partitions 字段会显示 p2021
```

### 5. type - 连接类型（性能从优到劣）

#### system - 系统表，只有一行

```sql
EXPLAIN SELECT @@version;
```

#### const - 常量查询，主键或唯一索引等值查询

```sql
EXPLAIN SELECT * FROM users WHERE id = 1;
```

#### eq_ref - 唯一性索引扫描

```sql
EXPLAIN SELECT u.name, c.name 
FROM users u 
JOIN cities c ON u.city_id = c.id 
WHERE u.id = 1;
```

#### ref - 非唯一性索引扫描

```sql
-- 假设 age 上有普通索引
EXPLAIN SELECT * FROM users WHERE age = 25;
```

#### range - 范围扫描

```sql
EXPLAIN SELECT * FROM users WHERE id BETWEEN 1 AND 100;
EXPLAIN SELECT * FROM users WHERE age IN (20, 25, 30);
EXPLAIN SELECT * FROM users WHERE name LIKE 'John%';
```

#### index - 索引全扫描

```sql
EXPLAIN SELECT id FROM users ORDER BY id;
```

#### ALL - 全表扫描（最差）

```sql
-- 没有索引或无法使用索引的查询
EXPLAIN SELECT * FROM users WHERE SUBSTRING(name, 1, 1) = 'J';
```

### 6. possible_keys - 可能使用的索引

```sql
-- 创建索引
CREATE INDEX idx_age ON users(age);
CREATE INDEX idx_name ON users(name);

-- 查询可能使用多个索引
EXPLAIN SELECT * FROM users WHERE age > 25 AND name LIKE 'John%';
-- possible_keys 会显示 idx_age, idx_name
```

### 7. key - 实际使用的索引

```sql
EXPLAIN SELECT * FROM users WHERE age = 25 AND name = 'John';
-- key 字段显示实际选择的索引
```

### 8. key_len - 索引使用长度

```sql
-- 复合索引
CREATE INDEX idx_age_name ON users(age, name);

EXPLAIN SELECT * FROM users WHERE age = 25;
-- key_len 显示只使用了 age 部分的长度

EXPLAIN SELECT * FROM users WHERE age = 25 AND name = 'John';
-- key_len 显示使用了完整索引的长度
```

### 9. ref - 索引比较的列

```sql
EXPLAIN SELECT u.*, c.name 
FROM users u 
JOIN cities c ON u.city_id = c.id;
-- ref 字段显示 u.city_id
```

### 10. rows - 估算扫描行数

```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
-- rows 字段显示估算需要扫描的行数
```

### 11. filtered - 过滤百分比

```sql
EXPLAIN SELECT * FROM users WHERE age > 25 AND name LIKE 'J%';
-- filtered 显示在应用 WHERE 条件后剩余的行百分比
```

### 12. Extra - 额外信息

#### Using index - 覆盖索引

```sql
CREATE INDEX idx_age_name ON users(age, name);
EXPLAIN SELECT age, name FROM users WHERE age > 25;
```

#### Using where - 使用 WHERE 过滤

```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
```

#### Using temporary - 使用临时表

```sql
EXPLAIN SELECT DISTINCT age FROM users;
EXPLAIN SELECT age, COUNT(*) FROM users GROUP BY age;
```

#### Using filesort - 使用文件排序

```sql
-- 没有对应索引的排序
EXPLAIN SELECT * FROM users ORDER BY age DESC;
```

#### Using join buffer - 使用连接缓冲

```sql
-- 没有可用索引的连接
EXPLAIN SELECT * FROM users u, cities c 
WHERE u.name = c.name;  -- 假设 name 上没有索引
```

#### Using index condition - 索引条件推送

```sql
EXPLAIN SELECT * FROM users 
WHERE age > 20 AND age < 30 AND name LIKE 'J%';
```

## EXPLAIN 格式变体

### 1. EXPLAIN EXTENDED

```sql
EXPLAIN EXTENDED SELECT * FROM users WHERE age > 25;
-- 然后执行
SHOW WARNINGS;
-- 可以看到优化后的查询语句
```

### 2. EXPLAIN FORMAT=JSON

```sql
EXPLAIN FORMAT=JSON SELECT u.name, c.name 
FROM users u 
JOIN cities c ON u.city_id = c.id 
WHERE u.age > 25;
```

### 3. EXPLAIN ANALYZE (MySQL 8.0+)

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 25;
-- 提供实际执行统计信息
```

## 实际优化示例

### 问题查询

```sql
-- 性能较差的查询
EXPLAIN SELECT * FROM users 
WHERE YEAR(created_at) = 2023 AND status = 'active';
```

### 优化方案

```sql
-- 1. 避免在索引列上使用函数
EXPLAIN SELECT * FROM users 
WHERE created_at >= '2023-01-01' 
AND created_at < '2024-01-01' 
AND status = 'active';

-- 2. 创建复合索引
CREATE INDEX idx_created_status ON users(created_at, status);

-- 3. 再次查看执行计划
EXPLAIN SELECT * FROM users 
WHERE created_at >= '2023-01-01' 
AND created_at < '2024-01-01' 
AND status = 'active';
```

## 性能分析检查清单

1. **type 字段检查**：避免 ALL（全表扫描）
2. **key 字段检查**：确保使用了合适的索引
3. **rows 字段检查**：扫描行数应该尽可能少
4. **Extra 字段检查**：
    - 好的：Using index, Using index condition
    - 需要注意：Using temporary, Using filesort
    - 差的：Using where（没有索引时）

## 常见优化技巧

### 1. 索引优化

```sql
-- 为经常查询的列创建索引
CREATE INDEX idx_age ON users(age);

-- 复合索引遵循最左前缀原则
CREATE INDEX idx_age_city_name ON users(age, city_id, name);
```

### 2. 查询重写

```sql
-- 原查询
SELECT * FROM users WHERE age + 10 > 35;

-- 优化后
SELECT * FROM users WHERE age > 25;
```

### 3. 子查询优化

```sql
-- 使用 EXISTS 替代 IN（某些情况下）
-- 原查询
SELECT * FROM users 
WHERE city_id IN (SELECT id FROM cities WHERE name = 'Beijing');

-- 优化后
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM cities c 
             WHERE c.id = u.city_id AND c.name = 'Beijing');
```

记住，EXPLAIN 只是分析工具，真正的性能测试还需要在实际数据量下进行基准测试。

**重点关注：**

1. **type 字段**：从 system 到 ALL，性能递减
2. **key 和 key_len**：确认索引使用情况
3. **rows 和 filtered**：评估查询效率
4. **Extra 字段**：发现潜在问题