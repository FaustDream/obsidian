# mysql索引（最左前缀原则了解吗，底层是用什么（B+树））

我来详细解释MySQL索引的最左前缀原则和底层实现机制。

## 最左前缀原则

**定义**：在使用联合索引时，查询必须从索引的最左侧字段开始匹配，才能有效利用索引。

**举例说明**：

```sql
-- 假设有联合索引 idx_abc(a, b, c)
CREATE INDEX idx_abc ON table_name(a, b, c);

-- 以下查询能使用索引：
SELECT * FROM table_name WHERE a = 1;                    -- 使用索引
SELECT * FROM table_name WHERE a = 1 AND b = 2;          -- 使用索引
SELECT * FROM table_name WHERE a = 1 AND b = 2 AND c = 3; -- 使用索引
SELECT * FROM table_name WHERE a = 1 AND c = 3;          -- 部分使用索引(只用a)

-- 以下查询不能使用索引：
SELECT * FROM table_name WHERE b = 2;                    -- 不使用索引
SELECT * FROM table_name WHERE c = 3;                    -- 不使用索引
SELECT * FROM table_name WHERE b = 2 AND c = 3;          -- 不使用索引
```

## B+树底层实现

### 为什么选择B+树？

1. **磁盘I/O优化**：B+树的非叶子节点只存储键值，不存储数据，能容纳更多的键值，减少树的高度
2. **范围查询友好**：叶子节点通过指针连接，形成有序链表，便于范围查询
3. **查询性能稳定**：所有数据都在叶子节点，查询路径长度一致

### B+树结构特点

```
        [10|20|30]           -- 非叶子节点（只存储键值）
       /   |   |   \
   [5|8] [12|15] [25] [35|40] -- 叶子节点（存储键值+数据/指针）
    ↓     ↓      ↓     ↓
   数据   数据    数据   数据
```

**关键特性**：

- 非叶子节点：只存储键值用于导航
- 叶子节点：存储完整的键值和数据记录（或指向数据的指针）
- 叶子节点之间通过指针相连，形成有序链表

### 索引类型

**聚簇索引（主键索引）**：

- 叶子节点直接存储完整的行数据
- 每个表只能有一个聚簇索引
- InnoDB中主键就是聚簇索引

**非聚簇索引（辅助索引）**：

- 叶子节点存储索引键值和主键值
- 需要回表查询获取完整数据

## 最左前缀原则的底层原理

### 联合索引的存储结构

```sql
-- 联合索引 idx_name_age_city(name, age, city)
-- 数据在B+树中的排序方式：
先按name排序 -> name相同时按age排序 -> age相同时按city排序

实际存储可能如下：
('Alice', 25, 'Beijing')
('Alice', 30, 'Shanghai')
('Bob', 20, 'Guangzhou')
('Bob', 25, 'Shenzhen')
```

### 为什么必须最左匹配？

因为B+树是按照联合索引字段的顺序进行排序的，只有从最左边开始匹配，才能利用这种有序性进行快速查找。

## 性能优化实践

### 1. 联合索引设计原则

```sql
-- 将区分度高的字段放在前面
-- 将经常用于WHERE条件的字段放在前面
CREATE INDEX idx_status_create_time ON orders(status, create_time);
```

### 2. 索引覆盖优化

```sql
-- 如果查询的字段都在索引中，可以避免回表
CREATE INDEX idx_user_info ON users(name, age, email);
SELECT name, age, email FROM users WHERE name = 'Alice';  -- 无需回表
```

### 3. 前缀索引

```sql
-- 对于长字符串字段，使用前缀索引
CREATE INDEX idx_email_prefix ON users(email(10));
```

## 扩展知识点

### 索引失效场景

1. **函数操作**：`WHERE UPPER(name) = 'ALICE'`
2. **类型转换**：`WHERE age = '25'`（age为整型）
3. **NOT条件**：`WHERE name != 'Alice'`
4. **OR条件**：`WHERE name = 'Alice' OR city = 'Beijing'`
5. **LIKE通配符开头**：`WHERE name LIKE '%Alice'`

### 索引监控

```sql
-- 查看索引使用情况
SHOW INDEX FROM table_name;
-- 分析查询执行计划
EXPLAIN SELECT * FROM table_name WHERE name = 'Alice';
```

这样的设计使得MySQL能够在海量数据中快速定位到目标记录，是关系型数据库高性能的关键所在。