# 如何查看SQL语句的执行效率

查看SQL语句执行效率有多种方法，我来为你详细介绍几种常用的方式：

## 1. 使用EXPLAIN分析执行计划

**MySQL示例：**

```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
```

**关键指标解读：**

- **type**: 连接类型（system > const > eq_ref > ref > range > index > ALL）
- **rows**: 预估扫描行数
- **key**: 使用的索引
- **Extra**: 额外信息（Using filesort、Using temporary等需要优化）

## 2. 查看执行时间

**MySQL方式：**

```sql
-- 开启分析
SET profiling = 1;

-- 执行SQL
SELECT * FROM orders WHERE order_date > '2024-01-01';

-- 查看执行时间
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

**Java应用层监控：**

```java
long startTime = System.currentTimeMillis();
// 执行SQL
long endTime = System.currentTimeMillis();
System.out.println("执行时间: " + (endTime - startTime) + "ms");
```

## 3. 数据库性能监控工具

**MySQL Performance Schema：**

```sql
-- 查看最慢的SQL
SELECT * FROM performance_schema.events_statements_summary_by_digest 
ORDER BY avg_timer_wait DESC LIMIT 10;
```

**慢查询日志：**

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- 1秒以上的查询
```

## 4. Java应用层监控

**使用Spring Boot Actuator：**

```java
@Component
public class SQLMonitor {
    
    @EventListener
    public void handleSQLExecution(SQLExecutionEvent event) {
        if (event.getExecutionTime() > 1000) {
            log.warn("慢SQL detected: {} ms, SQL: {}", 
                    event.getExecutionTime(), event.getSql());
        }
    }
}
```

**使用Druid监控：**

```properties
spring.datasource.druid.stat-view-servlet.enabled=true
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
spring.datasource.druid.filter.stat.enabled=true
```

## 5. 专业监控工具

**MyBatis监控：**

```xml
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="helperDialect" value="mysql"/>
    </plugin>
</plugins>
```

**使用P6Spy：**

```properties
# spy.properties
driverlist=com.mysql.cj.jdbc.Driver
realdriver=com.mysql.cj.jdbc.Driver
logMessageFormat=com.p6spy.engine.spy.appender.CustomLineFormat
customLogMessageFormat=%(currentTime) | %(executionTime) ms | %(sql)
```

## 6. 优化建议

**基于执行效率分析的优化策略：**

1. **索引优化**
    
    - 为WHERE、ORDER BY、GROUP BY字段建立索引
    - 避免索引失效（使用函数、OR条件等）
2. **SQL重写**
    
    - 避免SELECT *
    - 使用LIMIT限制返回数据量
    - 优化JOIN查询
3. **分页优化**
    
    ```sql
    -- 避免深分页
    SELECT * FROM users WHERE id > 1000 LIMIT 20;
    -- 而不是
    SELECT * FROM users LIMIT 1000, 20;
    ```
    

## 7. 实际应用示例

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    public List<User> getUsers(int page, int size) {
        String cacheKey = "users:" + page + ":" + size;
        
        // 先查缓存
        List<User> cachedUsers = (List<User>) redisTemplate.opsForValue().get(cacheKey);
        if (cachedUsers != null) {
            return cachedUsers;
        }
        
        // 数据库查询，监控执行时间
        long startTime = System.currentTimeMillis();
        List<User> users = userMapper.selectUsers(page * size, size);
        long executionTime = System.currentTimeMillis() - startTime;
        
        if (executionTime > 500) {
            log.warn("慢查询警告: {}ms, 分页参数: page={}, size={}", 
                    executionTime, page, size);
        }
        
        // 缓存结果
        redisTemplate.opsForValue().set(cacheKey, users, 5, TimeUnit.MINUTES);
        
        return users;
    }
}
```

通过这些方法，你可以全面监控和分析SQL执行效率，及时发现性能瓶颈并进行优化。在实际项目中，建议结合多种方式来综合评估SQL性能。