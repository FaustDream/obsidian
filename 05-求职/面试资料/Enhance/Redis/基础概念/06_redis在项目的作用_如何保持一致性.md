# redis在项目的作用、如何保持一致性

Redis在项目中主要有以下几个重要作用：

## Redis的主要作用

**缓存层**

- 将热点数据存储在内存中，大幅提升数据访问速度
- 减少数据库访问压力，提高系统整体性能
- 常见场景：用户信息缓存、商品信息缓存、页面缓存等

**分布式锁**

- 利用Redis的原子性操作实现分布式环境下的互斥锁
- 解决分布式系统中的并发控制问题
- 常用命令：SETNX、SET with NX和EX参数

**会话存储**

- 在分布式系统中存储用户会话信息
- 解决多服务器间会话共享问题
- 支持会话过期自动清理

**消息队列**

- 使用List、Stream等数据结构实现轻量级消息队列
- 支持发布订阅模式进行实时消息推送
- 适合处理异步任务和解耦系统组件

**计数器和限流**

- 利用原子性操作实现访问计数、点赞数等功能
- 基于滑动窗口或令牌桶算法实现接口限流

## 数据一致性保障策略

### 1. 缓存更新策略

**Cache Aside模式（最常用）**

```java
// 读取数据
public User getUser(Long userId) {
    // 先查缓存
    User user = redisTemplate.opsForValue().get("user:" + userId);
    if (user == null) {
        // 缓存未命中，查询数据库
        user = userMapper.selectById(userId);
        if (user != null) {
            // 写入缓存，设置过期时间
            redisTemplate.opsForValue().set("user:" + userId, user, 30, TimeUnit.MINUTES);
        }
    }
    return user;
}

// 更新数据
public void updateUser(User user) {
    // 先更新数据库
    userMapper.updateById(user);
    // 删除缓存，下次读取时重新加载
    redisTemplate.delete("user:" + user.getId());
}
```

**Write Through模式**

- 同时写入缓存和数据库
- 保证数据强一致性，但性能较差

**Write Behind模式**

- 先写缓存，异步写入数据库
- 性能最好，但可能丢失数据

### 2. 分布式锁保证一致性

```java
@Service
public class InventoryService {
    
    public boolean reduceStock(Long productId, int quantity) {
        String lockKey = "lock:product:" + productId;
        String lockValue = UUID.randomUUID().toString();
        
        try {
            // 获取分布式锁
            Boolean lockSuccess = redisTemplate.opsForValue()
                .setIfAbsent(lockKey, lockValue, 10, TimeUnit.SECONDS);
            
            if (!lockSuccess) {
                return false; // 获取锁失败
            }
            
            // 执行业务逻辑
            Integer currentStock = redisTemplate.opsForValue().get("stock:" + productId);
            if (currentStock >= quantity) {
                // 减库存
                redisTemplate.opsForValue().decrement("stock:" + productId, quantity);
                // 同步到数据库
                stockMapper.reduceStock(productId, quantity);
                return true;
            }
            return false;
            
        } finally {
            // 使用Lua脚本安全释放锁
            String luaScript = 
                "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                "return redis.call('del', KEYS[1]) else return 0 end";
            redisTemplate.execute(new DefaultRedisScript<>(luaScript, Long.class), 
                Arrays.asList(lockKey), lockValue);
        }
    }
}
```

### 3. 事务一致性处理

**本地事务 + 补偿机制**

```java
@Transactional
public void createOrder(Order order) {
    try {
        // 数据库操作
        orderMapper.insert(order);
        
        // 缓存操作
        redisTemplate.opsForValue().set("order:" + order.getId(), order);
        
        // 如果缓存操作失败，记录日志后续补偿
    } catch (Exception e) {
        // 回滚事务
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        // 清理可能的脏数据
        redisTemplate.delete("order:" + order.getId());
        throw e;
    }
}
```

### 4. 最终一致性方案

**基于MQ的异步同步**

```java
@Component
public class CacheRefreshListener {
    
    @RabbitListener(queues = "cache.refresh.queue")
    public void handleCacheRefresh(CacheRefreshMessage message) {
        switch (message.getType()) {
            case "USER_UPDATE":
                // 删除用户相关缓存
                redisTemplate.delete("user:" + message.getId());
                break;
            case "PRODUCT_UPDATE":
                // 刷新商品缓存
                Product product = productService.getById(message.getId());
                redisTemplate.opsForValue().set("product:" + message.getId(), product);
                break;
        }
    }
}
```

## 扩展知识点

**缓存穿透解决方案**

- 布隆过滤器预先过滤无效请求
- 缓存空值，设置较短过期时间

**缓存雪崩预防**

- 设置随机过期时间，避免同时失效
- 使用多级缓存架构
- 实现熔断降级机制

**缓存击穿处理**

- 热点数据永不过期
- 使用分布式锁控制数据库访问
- 实现异步刷新机制

**监控和运维**

- 监控缓存命中率、响应时间
- 设置内存使用告警
- 定期分析热点key和慢查询

这样的设计既保证了系统性能，又确保了数据的最终一致性，是生产环境中Redis应用的最佳实践。