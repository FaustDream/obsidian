# mysql慢查询调优

MySQL慢查询调优是数据库性能优化的核心技能。让我从识别问题到解决方案，系统地为你介绍这个重要话题。

## 慢查询识别与监控

**开启慢查询日志**

```sql
-- 查看慢查询配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 超过1秒的查询记录
```

**分析慢查询日志**

```bash
# 使用mysqldumpslow分析
mysqldumpslow -s c -t 10 /var/log/mysql/slow.log  # 按查询次数排序
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log  # 按查询时间排序
```

## 执行计划分析

**EXPLAIN详解**

```sql
EXPLAIN SELECT * FROM users WHERE age > 25 AND city = 'Shanghai';
```

关键字段解析：

- **type**: 连接类型（system > const > eq_ref > ref > range > index > ALL）
- **key**: 实际使用的索引
- **rows**: 预估扫描行数
- **Extra**: 额外信息（Using filesort、Using temporary需要关注）

## 索引优化策略

**索引设计原则**

```sql
-- 1. 联合索引遵循最左前缀原则
CREATE INDEX idx_user_age_city ON users(age, city, name);
-- 可以利用: age, (age,city), (age,city,name)
-- 不能利用: city, name, (city,name)

-- 2. 覆盖索引避免回表
CREATE INDEX idx_cover ON orders(user_id, status, order_time);
SELECT user_id, status, order_time FROM orders WHERE user_id = 123;
```

**索引失效场景**

```sql
-- 1. 函数操作导致索引失效
SELECT * FROM users WHERE DATE(create_time) = '2024-01-01';
-- 优化为:
SELECT * FROM users WHERE create_time >= '2024-01-01' AND create_time < '2024-01-02';

-- 2. 隐式类型转换
SELECT * FROM users WHERE phone = 13800138000;  -- phone是varchar
-- 优化为:
SELECT * FROM users WHERE phone = '13800138000';
```

## 查询语句优化

**子查询优化**

```sql
-- 低效的子查询
SELECT * FROM orders WHERE user_id IN (
    SELECT id FROM users WHERE city = 'Shanghai'
);

-- 优化为JOIN
SELECT o.* 
FROM orders o 
INNER JOIN users u ON o.user_id = u.id 
WHERE u.city = 'Shanghai';
```

**分页查询优化**

```sql
-- 传统分页（深度分页性能差）
SELECT * FROM orders ORDER BY id LIMIT 100000, 20;

-- 优化方案：使用游标
SELECT * FROM orders WHERE id > 100000 ORDER BY id LIMIT 20;

-- 或者使用延迟关联
SELECT o.* FROM orders o 
INNER JOIN (SELECT id FROM orders ORDER BY id LIMIT 100000, 20) t 
ON o.id = t.id;
```

## 数据库参数调优

**关键参数配置**

```ini
# my.cnf配置
[mysqld]
# 缓冲池大小（建议设置为物理内存的70-80%）
innodb_buffer_pool_size = 8G

# 查询缓存（MySQL 8.0已移除）
query_cache_size = 256M
query_cache_type = 1

# 连接相关
max_connections = 1000
max_connect_errors = 100

# InnoDB配置
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
```

## 架构层面优化

**读写分离**

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    
    // 写操作使用主库
    @Master
    public void updateUser(User user) {
        userMapper.updateUser(user);
    }
    
    // 读操作使用从库
    @Slave
    public User getUserById(Long id) {
        return userMapper.selectById(id);
    }
}
```

**分库分表**

```sql
-- 水平分表示例
CREATE TABLE orders_2024_01 LIKE orders;
CREATE TABLE orders_2024_02 LIKE orders;
-- 根据时间或ID进行分表路由
```

## 缓存策略

**多级缓存架构**

```java
// 应用级缓存
@Cacheable(value = "user", key = "#id")
public User getUserById(Long id) {
    return userMapper.selectById(id);
}

// Redis缓存
@Service
public class UserCacheService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public User getUserFromCache(Long id) {
        String key = "user:" + id;
        return (User) redisTemplate.opsForValue().get(key);
    }
}
```

## 监控与告警

**性能监控指标**

```sql
-- 查看慢查询统计
SHOW GLOBAL STATUS LIKE 'Slow_queries';

-- 查看连接数
SHOW GLOBAL STATUS LIKE 'Connections';
SHOW GLOBAL STATUS LIKE 'Threads_connected';

-- 查看InnoDB状态
SHOW ENGINE INNODB STATUS;
```

**常用监控工具**

- **Prometheus + Grafana**: 监控MySQL性能指标
- **pt-query-digest**: 分析慢查询日志
- **MySQL Workbench**: 可视化性能分析

## 实战案例

假设有一个订单查询慢的问题：

```sql
-- 原始慢查询
SELECT * FROM orders o 
LEFT JOIN users u ON o.user_id = u.id 
WHERE o.status = 'pending' 
AND o.create_time >= '2024-01-01' 
ORDER BY o.create_time DESC 
LIMIT 20;
```

**优化步骤**：

1. 分析执行计划，发现缺少复合索引
2. 创建索引：`CREATE INDEX idx_orders_status_time ON orders(status, create_time)`
3. 如果还需要用户信息，考虑先查订单ID，再关联用户
4. 添加适当的缓存层

慢查询调优是一个持续的过程，需要结合业务场景、数据量特点来制定具体的优化策略。关键是要建立完善的监控体系，及时发现和解决性能问题。