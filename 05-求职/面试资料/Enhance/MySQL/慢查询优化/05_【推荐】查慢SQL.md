# 查慢SQL

我来详细解释一下查慢SQL的方法和相关知识点。

## 什么是慢SQL

慢SQL是指执行时间超过预期阈值的SQL语句，通常会影响系统性能和用户体验。

## 查慢SQL的主要方法

### 1. 数据库层面查询

**MySQL慢查询日志**

```sql
-- 查看慢查询日志是否开启
SHOW VARIABLES LIKE 'slow_query_log';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 1;

-- 设置慢查询阈值（秒）
SET GLOBAL long_query_time = 2;

-- 查看慢查询日志文件位置
SHOW VARIABLES LIKE 'slow_query_log_file';
```

**实时查看正在执行的慢SQL**

```sql
-- 查看当前运行的查询
SHOW PROCESSLIST;

-- 查看详细的查询信息
SELECT * FROM information_schema.PROCESSLIST 
WHERE COMMAND != 'Sleep' AND TIME > 5;
```

### 2. 应用层面监控

**Spring Boot + MyBatis示例**

```java
// 配置慢SQL监控
@Configuration
public class DruidConfig {
    
    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        // 其他配置...
        
        // 配置慢SQL监控
        dataSource.setFilters("stat,wall,log4j");
        dataSource.setConnectionProperties("druid.stat.slowSqlMillis=2000");
        return dataSource;
    }
}

// 自定义拦截器监控SQL执行时间
@Intercepts({
    @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
    @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
})
public class SlowSqlInterceptor implements Interceptor {
    
    private static final long SLOW_SQL_THRESHOLD = 2000; // 2秒
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        try {
            return invocation.proceed();
        } finally {
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;
            
            if (executionTime > SLOW_SQL_THRESHOLD) {
                MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
                String sqlId = mappedStatement.getId();
                
                log.warn("慢SQL检测 - SQL ID: {}, 执行时间: {}ms", sqlId, executionTime);
                
                // 可以发送告警或记录到监控系统
                sendSlowSqlAlert(sqlId, executionTime);
            }
        }
    }
}
```

### 3. 监控工具

**使用APM工具**

```java
// 集成SkyWalking
@Component
public class SqlPerformanceMonitor {
    
    @Trace
    public void monitorSqlExecution(String sql, long executionTime) {
        if (executionTime > 2000) {
            // 记录到链路追踪系统
            Tags.of("sql.slow", "true")
                .and("execution.time", String.valueOf(executionTime))
                .and("sql.statement", sql);
        }
    }
}
```

## 分析慢SQL的步骤

### 1. 使用EXPLAIN分析执行计划

```sql
EXPLAIN SELECT * FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.create_time > '2024-01-01';
```

**关键字段分析**

- `type`: 连接类型（ALL最差，const最好）
- `key`: 实际使用的索引
- `rows`: 扫描的行数
- `Extra`: 额外信息（Using filesort, Using temporary需要优化）

### 2. 查看索引使用情况

```sql
-- 查看索引使用统计
SHOW INDEX FROM table_name;

-- 查看未使用的索引
SELECT * FROM sys.schema_unused_indexes;
```

## 慢SQL优化策略

### 1. 索引优化

```java
// 实体类添加索引注解
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_create_time", columnList = "create_time"),
    @Index(name = "idx_email", columnList = "email", unique = true),
    @Index(name = "idx_status_create_time", columnList = "status, create_time")
})
public class User {
    // 字段定义...
}
```

### 2. 查询优化

```java
// 避免SELECT *
@Query("SELECT u.id, u.name, u.email FROM User u WHERE u.status = :status")
List<UserDTO> findActiveUsers(@Param("status") String status);

// 使用分页查询
@Query("SELECT u FROM User u WHERE u.createTime > :startTime")
Page<User> findUsersByCreateTime(@Param("startTime") LocalDateTime startTime, 
                                Pageable pageable);

// 批量操作优化
@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.id IN :ids")
int updateUserStatus(@Param("status") String status, @Param("ids") List<Long> ids);
```

### 3. 缓存策略

```java
@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    @Cacheable(value = "userList", key = "#status")
    public List<User> findByStatus(String status) {
        return userRepository.findByStatus(status);
    }
}
```

## 相关扩展知识

### 1. 数据库连接池配置

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    public HikariDataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        // 检测慢SQL
        config.setLeakDetectionThreshold(60000);
        
        return new HikariDataSource(config);
    }
}
```

### 2. 分库分表场景下的慢SQL

```java
// 使用ShardingSphere处理分库分表
@Configuration
public class ShardingConfig {
    
    @Bean
    public DataSource dataSource() {
        // 分片规则配置
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        
        // 配置慢SQL阈值
        Properties props = new Properties();
        props.setProperty("sql.show", "true");
        props.setProperty("sql.slow.milliseconds", "2000");
        
        return ShardingDataSourceFactory.createDataSource(
            createDataSourceMap(), shardingRuleConfig, props);
    }
}
```

### 3. 读写分离优化

```java
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    // 读操作使用从库
    @Transactional(readOnly = true)
    public List<Order> findOrdersByUserId(Long userId) {
        return orderRepository.findByUserId(userId);
    }
    
    // 写操作使用主库
    @Transactional
    public Order createOrder(Order order) {
        return orderRepository.save(order);
    }
}
```

## 最佳实践建议

1. **预防为主**: 在开发阶段就要考虑SQL性能，做好索引设计
2. **定期监控**: 建立慢SQL监控告警机制
3. **及时优化**: 发现慢SQL后要及时分析和优化
4. **压测验证**: 在生产环境上线前做好压力测试
5. **持续改进**: 根据业务发展不断调整数据库设计和查询策略

这些方法和知识点可以帮助你在面试中全面回答慢SQL相关问题，同时在实际工作中有效解决性能问题。