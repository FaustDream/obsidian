## 1. 准备测试数据

首先创建几个测试表来演示不同的执行计划场景：

```sql
-- 用户表
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    age INT,
    city VARCHAR(30),
    INDEX idx_age (age),
    INDEX idx_city (city)
);

-- 订单表
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    amount DECIMAL(10,2),
    order_date DATE,
    status VARCHAR(20),
    INDEX idx_user_id (user_id),
    INDEX idx_order_date (order_date),
    INDEX idx_status (status)
);

-- 插入测试数据
INSERT INTO users (name, age, city) VALUES
('张三', 25, '北京'),
('李四', 30, '上海'),
('王五', 28, '广州'),
('赵六', 35, '深圳');

INSERT INTO orders (user_id, amount, order_date, status) VALUES
(1, 100.00, '2024-01-15', 'completed'),
(2, 250.50, '2024-01-16', 'pending'),
(1, 80.75, '2024-01-17', 'completed'),
(3, 300.00, '2024-01-18', 'cancelled');
```

## 2. EXPLAIN输出字段详解

### 2.1 基本EXPLAIN示例

```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
```

典型输出：

```
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | users | NULL       | range | idx_age       | idx_age | 5       | NULL |    2 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
```

### 2.2 各字段含义详解

#### **id** - 查询序号
- **含义**：SELECT语句的序号，表示执行的顺序
- **规则**：
    - 数字越大越先执行
    - 相同id从上到下执行
    - NULL表示结果集，通常出现在UNION操作中
#### **select_type** - 查询类型
- **SIMPLE**：简单SELECT，不使用UNION或子查询
- **PRIMARY**：最外层的SELECT
- **SUBQUERY**：子查询中的第一个SELECT
- **DERIVED**：派生表的SELECT（FROM子句中的子查询）
- **UNION**：UNION中的第二个或后续SELECT
- **UNION RESULT**：UNION的结果
#### **table** - 表名
- 显示这一行数据是关于哪张表的
- 可能显示别名或临时表名
#### **partitions** - 分区信息
- 显示查询涉及的分区（如果表已分区）
#### **type** - 连接类型（性能从好到差）
- **system**：表只有一行记录（系统表）
- **const**：通过索引一次就找到，用于PRIMARY KEY或UNIQUE索引
- **eq_ref**：唯一性索引扫描，对于每个索引键，表中只有一条记录匹配
- **ref**：非唯一性索引扫描，返回匹配某个单独值的所有行
- **range**：索引范围扫描，如BETWEEN、>、<等
- **index**：全索引扫描
- **ALL**：全表扫描（性能最差）
#### **possible_keys** - 可能使用的索引
- MySQL能使用哪些索引来查找该表中的记录
#### **key** - 实际使用的索引
- 实际从possible_keys中选择使用的索引
- 如果为NULL，则没有使用索引
#### **key_len** - 索引长度
- 表示索引中使用的字节数
- 在不损失精确性的情况下，长度越短越好
#### **ref** - 索引比较的列
- 显示索引的哪一列被使用了
- 如果是常数，则显示const
#### **rows** - 扫描行数
- MySQL认为必须检索的行数
- 这是一个估计值
#### **filtered** - 过滤百分比
- 表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例
#### **Extra** - 额外信息
- **Using where**：使用WHERE子句过滤结果
- **Using index**：使用覆盖索引
- **Using filesort**：使用文件排序
- **Using temporary**：使用临时表
- **Using index condition**：使用索引条件下推

## 3. 复杂查询执行顺序案例

### 3.1 JOIN查询执行计划

```sql
EXPLAIN SELECT u.name, o.amount, o.order_date 
FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.age > 25 AND o.status = 'completed'
ORDER BY o.order_date DESC;
```

执行计划分析：

```
+----+-------------+-------+-------+------------------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | type  | possible_keys    | key     | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+-------+------------------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | u     | range | PRIMARY,idx_age  | idx_age | NULL |    2 |   100.00 | Using where                                        |
|  1 | SIMPLE      | o     | ref   | idx_user_id      | idx_user | u.id |    1 |    33.33 | Using where                                        |
+----+-------------+-------+-------+------------------+---------+------+------+----------+----------------------------------------------------+
```

**执行顺序**：

1. 首先扫描users表（驱动表），使用idx_age索引找到age > 25的记录
2. 对每个符合条件的user记录，通过idx_user_id在orders表中查找匹配的记录
3. 应用WHERE条件过滤status = 'completed'的订单

### 3.2 子查询执行计划

```sql
EXPLAIN SELECT * FROM users 
WHERE id IN (
    SELECT user_id FROM orders WHERE amount > 200
);
```

执行计划：

```
+----+--------------+-------------+--------+---------------+-------------+---------+-----------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key         | key_len | ref       | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+-------------+---------+-----------+------+----------+-------------+
|  1 | SIMPLE       | orders      | ALL    | NULL          | NULL        | NULL    | NULL      |    4 |    25.00 | Using where |
|  1 | SIMPLE       | users       | eq_ref | PRIMARY       | PRIMARY     | 4       | orders.user_id | 1 |   100.00 | NULL        |
+----+--------------+-------------+--------+---------------+-------------+---------+-----------+------+----------+-------------+
```

**执行顺序**：

1. MySQL将IN子查询转换为半连接
2. 扫描orders表，找到amount > 200的记录
3. 使用user_id在users表的主键上进行eq_ref查找

### 3.3 UNION查询执行计划

```sql
EXPLAIN 
SELECT name, age FROM users WHERE city = '北京'
UNION
SELECT name, age FROM users WHERE city = '上海';
```

执行计划：

```
+------+--------------+------------+------+---------------+----------+------+-------+----------+-----------------+
| id   | select_type  | table      | type | possible_keys | key      | ref  | rows  | filtered | Extra           |
+------+--------------+------------+------+---------------+----------+------+-------+----------+-----------------+
|    1 | PRIMARY      | users      | ref  | idx_city      | idx_city | const|     1 |   100.00 | Using where     |
|    2 | UNION        | users      | ref  | idx_city      | idx_city | const|     1 |   100.00 | Using where     |
| NULL | UNION RESULT | <union1,2> | ALL  | NULL          | NULL     | NULL | NULL  |     NULL | Using temporary |
+------+--------------+------------+------+---------------+----------+------+-------+----------+-----------------+
```

**执行顺序**：

1. 执行第一个SELECT（id=1，PRIMARY）
2. 执行第二个SELECT（id=2，UNION）
3. 合并结果（UNION RESULT），使用临时表去重

## 4. 性能优化建议

### 4.1 根据type字段优化

```sql
-- 差的查询（type=ALL，全表扫描）
EXPLAIN SELECT * FROM users WHERE name LIKE '%张%';

-- 优化后（创建索引）
CREATE INDEX idx_name ON users(name);
-- 但LIKE '%张%'仍然无法使用索引，需要改为：
EXPLAIN SELECT * FROM users WHERE name LIKE '张%';
```

### 4.2 根据Extra字段优化

```sql
-- 出现Using filesort（需要排序优化）
EXPLAIN SELECT * FROM orders ORDER BY amount;

-- 优化：创建复合索引
CREATE INDEX idx_amount_date ON orders(amount, order_date);

-- 覆盖索引优化（Extra显示Using index）
EXPLAIN SELECT user_id, amount FROM orders WHERE user_id = 1;
```

## 5. 实用查询技巧

### 5.1 使用EXPLAIN FORMAT=JSON获取更详细信息

```sql
EXPLAIN FORMAT=JSON 
SELECT u.name, COUNT(o.id) as order_count
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
GROUP BY u.id, u.name;
```

### 5.2 使用EXPLAIN ANALYZE（MySQL 8.0+）

```sql
EXPLAIN ANALYZE 
SELECT * FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.age BETWEEN 25 AND 35;
```

这会显示实际执行时间和实际处理的行数。

## 6. 执行计划读取要点

1. **从上到下，从右到左**：一般按id从大到小，相同id从上到下读取
2. **重点关注type列**：避免ALL和index类型
3. **观察rows列**：估算查询的开销
4. **分析Extra列**：识别性能瓶颈
5. **检查key列**：确保重要查询使用了合适的索引

通过理解这些参数和执行顺序，可以更好地优化SQL查询性能。