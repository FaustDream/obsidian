# 慢sql治理经验

## 慢SQL治理经验

### 1. 慢SQL识别与监控

**监控工具建立**

- 使用MySQL的慢查询日志（slow query log）设置合理的阈值（通常1-2秒）
- 集成APM工具如SkyWalking、Pinpoint监控应用层SQL执行时间
- 部署数据库监控系统如Prometheus + Grafana实时监控数据库性能指标

**关键监控指标**

- SQL执行时间、扫描行数、返回行数
- 数据库连接数、锁等待时间
- CPU使用率、内存使用率、磁盘I/O

### 2. 慢SQL分析诊断

**EXPLAIN执行计划分析**

```sql
EXPLAIN SELECT * FROM orders o 
JOIN customers c ON o.customer_id = c.id 
WHERE o.order_date > '2023-01-01';
```

重点关注：

- type字段：避免ALL全表扫描
- key字段：确认是否使用索引
- rows字段：预估扫描行数
- Extra字段：关注filesort、temporary等

**慢SQL分类治理**

- **大表全表扫描**：添加合适索引，优化WHERE条件
- **深度分页**：使用游标分页或限制分页深度
- **复杂JOIN**：优化JOIN顺序，添加合适的复合索引
- **函数索引失效**：避免在WHERE条件中使用函数

### 3. 索引优化策略

**索引设计原则**

```sql
-- 复合索引遵循最左前缀原则
CREATE INDEX idx_order_date_status ON orders(order_date, status);

-- 覆盖索引减少回表
CREATE INDEX idx_customer_name_email ON customers(name, email);
```

**索引监控维护**

- 定期分析索引使用情况，删除冗余索引
- 监控索引碎片率，及时重建索引
- 避免过度索引影响写入性能

### 4. SQL重写优化

**常见优化技巧**

```sql
-- 子查询改写为JOIN
-- 优化前
SELECT * FROM orders WHERE customer_id IN 
(SELECT id FROM customers WHERE city = 'Beijing');

-- 优化后
SELECT o.* FROM orders o 
JOIN customers c ON o.customer_id = c.id 
WHERE c.city = 'Beijing';

-- 避免SELECT *
SELECT id, name, email FROM customers WHERE status = 'active';

-- 使用LIMIT限制结果集
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

### 5. 数据库参数调优

**关键参数优化**

```sql
-- 增加缓冲池大小
SET GLOBAL innodb_buffer_pool_size = 8G;

-- 调整查询缓存
SET GLOBAL query_cache_size = 256M;

-- 优化连接参数
SET GLOBAL max_connections = 1000;
SET GLOBAL wait_timeout = 300;
```

### 6. 架构层面优化

**读写分离**

- 主库处理写操作，从库处理读操作
- 使用中间件如MyCat、Sharding-JDBC实现透明读写分离

**分库分表**

- 垂直分库：按业务模块划分
- 水平分表：按数据量或时间维度分表
- 使用分布式中间件处理跨库查询

**缓存策略**

```java
@Service
public class OrderService {
    
    @Cacheable(value = "orders", key = "#orderId")
    public Order getOrder(Long orderId) {
        return orderMapper.selectById(orderId);
    }
    
    @CacheEvict(value = "orders", key = "#order.id")
    public void updateOrder(Order order) {
        orderMapper.updateById(order);
    }
}
```

### 7. 应用层优化

**连接池优化**

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        return new HikariDataSource(config);
    }
}
```

**批量操作优化**

```java
// 批量插入
@Transactional
public void batchInsert(List<Order> orders) {
    orderMapper.insertBatch(orders);
}

// 分页查询避免深度分页
public Page<Order> getOrdersByPage(int page, int size) {
    if (page > 100) {
        throw new IllegalArgumentException("分页深度过大");
    }
    return orderMapper.selectPage(page, size);
}
```

### 8. 治理流程与规范

**SQL审核流程**

- 开发阶段：IDE插件进行SQL规范检查
- 测试阶段：自动化测试包含SQL性能测试
- 上线前：DBA进行SQL审核，执行计划分析

**监控告警机制**

- 设置慢SQL告警阈值
- 建立SQL性能下降自动告警
- 定期生成SQL性能报告

### 9. 相关衍生知识

**MySQL存储引擎特性**

- InnoDB支持事务、行级锁、外键约束
- MyISAM表级锁，适合读多写少场景
- Memory引擎数据存储在内存中，适合临时表

**数据库锁机制**

- 行锁vs表锁的性能影响
- 死锁产生原因与避免策略
- 锁等待超时处理

**分布式事务处理**

- 两阶段提交（2PC）
- 三阶段提交（3PC）
- 最终一致性方案

通过系统化的慢SQL治理，可以显著提升数据库性能，保障系统稳定运行。关键是要建立完善的监控体系，制定规范的优化流程，并持续跟踪优化效果。