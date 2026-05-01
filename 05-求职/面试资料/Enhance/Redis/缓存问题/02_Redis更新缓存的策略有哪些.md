# Redis更新缓存的策略有哪些？

Redis更新缓存的策略主要有以下几种：

## 1. Cache Aside（旁路缓存）

这是最常用的缓存策略：

- **读取**：先查缓存，缓存miss则查数据库，然后将数据写入缓存
- **更新**：==先更新数据库，然后删除缓存（而不是更新缓存）==

```java
// 读取示例
public User getUser(Long id) {
    User user = redisTemplate.opsForValue().get("user:" + id);
    if (user == null) {
        user = userRepository.findById(id);
        if (user != null) {
            redisTemplate.opsForValue().set("user:" + id, user, 30, TimeUnit.MINUTES);
        }
    }
    return user;
}

// 更新示例
public void updateUser(User user) {
    userRepository.save(user);
    redisTemplate.delete("user:" + user.getId());
}
```

## 2. Write Through（写穿透）

同时更新缓存和数据库：

- 应用程序只需要与缓存交互
- 缓存负责与数据库的同步
- 保证数据一致性，但写入性能较差

## 3. Write Behind（写回/异步写）

先更新缓存，异步更新数据库：

- 写入性能最好
- 但存在数据丢失风险
- 适用于对一致性要求不高的场景

## 4. Refresh Ahead（预刷新）

在缓存过期前主动刷新：

- 通过TTL监控，在过期前异步更新
- 避免缓存穿透问题
- 保证热点数据始终在缓存中

## 关键考虑因素

### 为什么删除而不是更新缓存？

1. **并发问题**：并发更新可能导致数据不一致
2. **性能考虑**：删除操作更轻量
3. **懒加载**：按需加载，避免无效更新

### 双写一致性问题

经典的数据库和缓存不一致场景：

```java
// 可能的问题序列
// 线程A：更新数据库
// 线程B：查询缓存miss，查询数据库（旧值）
// 线程A：删除缓存
// 线程B：将旧值写入缓存
```

**解决方案**：

- 延时双删策略
- 使用分布式锁
- 基于消息队列的异步处理

### 缓存雪崩、击穿、穿透的预防

- **雪崩**：设置随机过期时间
- **击穿**：使用分布式锁或热点数据永不过期
- **穿透**：布隆过滤器 + 空值缓存

## 实际应用建议

1. **读多写少场景**：使用Cache Aside
2. **强一致性要求**：考虑Write Through
3. **高并发写入**：可考虑Write Behind
4. **热点数据**：结合Refresh Ahead策略

选择合适的策略需要根据业务场景的一致性要求、性能需求和数据特征来决定。