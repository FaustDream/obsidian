# 保证Redis和数据库的一致性常见的方案有哪些？

保证Redis和数据库一致性是分布式系统中的经典问题。以下是几种常见的解决方案：

## 1. 缓存更新策略

### Cache Aside模式（最常用）

- **读取流程**：先查缓存，缓存未命中再查数据库，然后将数据写入缓存
- **写入流程**：先更新数据库，成功后删除缓存
- **优点**：逻辑简单，适用性广
- **缺点**：存在短暂不一致窗口期

### Write Through模式

- 同时写入缓存和数据库
- 写入操作较慢，但一致性较好

### Write Behind模式

- 先写缓存，异步写入数据库
- 性能好但可能丢失数据

## 2. 双写一致性方案

### 先删缓存，再更新数据库

```java
// 伪代码示例
public void updateUser(User user) {
    // 1. 先删除缓存
    redisTemplate.delete("user:" + user.getId());
    
    // 2. 更新数据库
    userDao.update(user);
    
    // 3. 延时双删（可选）
    Thread.sleep(500);
    redisTemplate.delete("user:" + user.getId());
}
```

### 先更新数据库，再删缓存（推荐）

```java
public void updateUser(User user) {
    // 1. 更新数据库
    userDao.update(user);
    
    // 2. 删除缓存
    redisTemplate.delete("user:" + user.getId());
}
```

## 3. 分布式事务方案

### 基于MQ的最终一致性

```java
@Transactional
public void updateUser(User user) {
    // 1. 更新数据库
    userDao.update(user);
    
    // 2. 发送MQ消息
    rabbitTemplate.send("cache.delete", user.getId());
}

// 消费者处理缓存删除
@RabbitListener(queues = "cache.delete")
public void deleteCacheHandler(Long userId) {
    redisTemplate.delete("user:" + userId);
}
```

### 基于Canal监听binlog

- 监听MySQL的binlog变化
- 异步更新Redis缓存
- 实现数据库和缓存的准实时同步

## 4. 读写分离场景的解决方案

### 延时双删策略

```java
public void updateUser(User user) {
    // 1. 删除缓存
    redisTemplate.delete("user:" + user.getId());
    
    // 2. 更新主库
    userDao.update(user);
    
    // 3. 延时删除缓存（解决主从延迟问题）
    CompletableFuture.runAsync(() -> {
        try {
            Thread.sleep(1000); // 主从同步延迟时间
            redisTemplate.delete("user:" + user.getId());
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}
```

## 5. 强一致性方案

### 分布式锁

```java
public User getUser(Long userId) {
    String lockKey = "lock:user:" + userId;
    RLock lock = redisson.getLock(lockKey);
    
    try {
        lock.lock(5, TimeUnit.SECONDS);
        
        // 双重检查
        User user = redisTemplate.get("user:" + userId);
        if (user == null) {
            user = userDao.findById(userId);
            if (user != null) {
                redisTemplate.set("user:" + userId, user, 300);
            }
        }
        return user;
    } finally {
        lock.unlock();
    }
}
```

## 6. 实际项目中的最佳实践

### 组合方案

```java
@Service
public class UserService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private UserDao userDao;
    
    public void updateUser(User user) {
        String cacheKey = "user:" + user.getId();
        
        try {
            // 1. 更新数据库
            userDao.update(user);
            
            // 2. 删除缓存
            redisTemplate.delete(cacheKey);
            
        } catch (Exception e) {
            // 3. 补偿机制 - 记录失败日志，后续异步处理
            log.error("缓存删除失败，userId: {}", user.getId(), e);
            // 可以发送到死信队列或重试队列
        }
    }
    
    public User getUser(Long userId) {
        String cacheKey = "user:" + userId;
        
        // 1. 查缓存
        User user = (User) redisTemplate.opsForValue().get(cacheKey);
        if (user != null) {
            return user;
        }
        
        // 2. 查数据库
        user = userDao.findById(userId);
        if (user != null) {
            // 3. 设置缓存，添加随机过期时间防止缓存雪崩
            int randomExpire = 300 + new Random().nextInt(60);
            redisTemplate.opsForValue().set(cacheKey, user, randomExpire, TimeUnit.SECONDS);
        }
        
        return user;
    }
}
```

## 选择建议

1. **一般业务**：Cache Aside + 先更新数据库再删缓存
2. **高并发场景**：引入MQ实现最终一致性
3. **强一致性要求**：使用分布式锁，但会影响性能
4. **大数据量场景**：考虑Canal同步binlog方案

每种方案都有其适用场景和权衡，需要根据具体业务需求、并发量、一致性要求来选择合适的方案。