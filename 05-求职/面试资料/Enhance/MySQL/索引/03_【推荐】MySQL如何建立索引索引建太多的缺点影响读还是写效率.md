# MySQL如何建立索引？索引建太多的缺点？影响读还是写效率？

## MySQL索引建立方式

MySQL中建立索引主要有以下几种方法：

### 1. 创建表时同时创建索引

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100),
    age INT,
    INDEX idx_name (name),
    INDEX idx_email (email),
    INDEX idx_age_name (age, name)  -- 复合索引
);
```

### 2. 在已有表上创建索引

```sql
-- 普通索引
CREATE INDEX idx_name ON users(name);

-- 唯一索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- 复合索引
CREATE INDEX idx_age_name ON users(age, name);
```

### 3. 使用ALTER TABLE添加索引

```sql
ALTER TABLE users ADD INDEX idx_name (name);
ALTER TABLE users ADD UNIQUE INDEX idx_email (email);
```

### 4. 主键和唯一约束自动创建索引

```sql
-- 主键自动创建聚簇索引
ALTER TABLE users ADD PRIMARY KEY (id);

-- 唯一约束自动创建唯一索引
ALTER TABLE users ADD UNIQUE (email);
```

## 索引过多的缺点

### 1. 存储空间开销

- 每个索引都需要额外的磁盘空间存储
- B+树结构需要维护索引页面
- 复合索引占用空间更大

### 2. 维护成本增加

- 每次数据变更时，所有相关索引都需要更新
- 索引碎片化需要定期维护
- 增加了数据库的复杂性

### 3. 查询优化器选择困难

- 索引过多可能导致优化器选择不当
- 可能选择了非最优的索引执行计划
- 增加了查询分析的时间

## 主要影响写效率

索引过多**主要影响写操作效率**，具体表现为：

### 写操作影响（INSERT/UPDATE/DELETE）：

```sql
-- 每次插入时，需要维护所有索引
INSERT INTO users (name, email, age) VALUES ('张三', 'zhang@email.com', 25);
-- 这条语句需要同时更新：
-- 1. 主键索引
-- 2. name索引  
-- 3. email索引
-- 4. age相关的复合索引
```

**影响程度**：

- INSERT：需要在所有索引中插入新记录
- UPDATE：如果更新索引列，需要删除旧索引记录并插入新记录
- DELETE：需要从所有索引中删除对应记录

### 读操作影响较小：

- 查询时优化器只会选择1-2个最优索引
- 不会使用所有索引
- 合适的索引显著提升查询效率

## 相关扩展知识

### 1. 索引选择原则

- 经常出现在WHERE条件中的列
- 经常用于JOIN的列
- 经常用于ORDER BY的列
- 选择性高的列（重复值少）

### 2. 不建议创建索引的情况

- 数据量很小的表
- 经常进行大量写操作的表
- 选择性很低的列（如性别字段）

### 3. 索引优化策略

```sql
-- 使用覆盖索引减少回表
CREATE INDEX idx_name_age ON users(name, age);
SELECT name, age FROM users WHERE name = '张三';

-- 前缀索引节省空间
CREATE INDEX idx_email_prefix ON users(email(10));
```

### 4. 监控索引使用情况

```sql
-- 查看索引使用统计
SHOW INDEX FROM users;

-- 查看未使用的索引
SELECT * FROM sys.schema_unused_indexes;
```

合理的索引设计需要在查询性能和写入性能之间找到平衡点，通常建议根据实际业务场景和查询模式来设计索引策略。