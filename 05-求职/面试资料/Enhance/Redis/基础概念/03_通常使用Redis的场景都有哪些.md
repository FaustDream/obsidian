# 通常使用Redis的场景都有哪些？

Redis作为高性能的内存数据库，在实际开发中有很多应用场景，我来详细介绍一下：

## 主要应用场景

**1. 缓存系统**

- **数据库查询缓存**：将频繁查询的数据库结果存储在Redis中，减少数据库压力
- **页面缓存**：缓存静态页面内容，提高网站响应速度
- **对象缓存**：缓存计算结果复杂的对象，避免重复计算

```java
// 示例：查询用户信息时先从Redis获取
String userKey = "user:" + userId;
String userJson = redisTemplate.opsForValue().get(userKey);
if (userJson == null) {
    User user = userService.getUserFromDB(userId);
    redisTemplate.opsForValue().set(userKey, JSON.toJSONString(user), 30, TimeUnit.MINUTES);
    return user;
}
return JSON.parseObject(userJson, User.class);
```

**2. 会话存储(Session Store)**

- 在分布式系统中存储用户会话信息
- 支持多台服务器共享session数据
- 解决负载均衡环境下的session一致性问题

**3. 分布式锁**

- 利用Redis的原子性操作实现分布式锁
- 防止并发环境下的数据竞争问题

```java
// 使用SET NX EX实现分布式锁
public boolean tryLock(String key, String value, int expireSeconds) {
    String result = redisTemplate.execute((RedisCallback<String>) connection -> {
        return connection.set(key.getBytes(), value.getBytes(), 
                             Expiration.seconds(expireSeconds), 
                             RedisStringCommands.SetOption.SET_IF_ABSENT);
    });
    return "OK".equals(result);
}
```

**4. 消息队列**

- **List结构**：实现简单的消息队列（LPUSH + BRPOP）
- **Stream结构**：实现更复杂的消息流处理
- **发布订阅**：实现实时消息推送

**5. 计数器和统计**

- **访问量统计**：网站PV、UV统计
- **点赞数、评论数**：社交媒体功能
- **限流控制**：API调用次数限制

```java
// 实现接口限流
public boolean isAllowed(String userId, int limit, int window) {
    String key = "rate_limit:" + userId;
    Long count = redisTemplate.opsForValue().increment(key);
    if (count == 1) {
        redisTemplate.expire(key, window, TimeUnit.SECONDS);
    }
    return count <= limit;
}
```

**6. 排行榜系统**

- 利用Sorted Set实现各种排行榜
- 游戏积分排行、商品销量排行等

```java
// 更新用户分数并获取排行
redisTemplate.opsForZSet().add("leaderboard", userId, score);
Set<String> topUsers = redisTemplate.opsForZSet().reverseRange("leaderboard", 0, 9);
```

**7. 地理位置服务**

- 使用GEO数据类型存储地理位置信息
- 实现附近的人、附近的商家等功能

**8. 实时数据处理**

- **购物车**：存储用户购物车信息
- **最近浏览**：记录用户浏览历史
- **在线用户**：维护在线用户列表

## 为什么选择Redis？

**性能优势**：

- 纯内存操作，读写速度极快
- 单线程模型避免了锁竞争
- 支持数据持久化

**数据结构丰富**：

- String、Hash、List、Set、Sorted Set
- HyperLogLog、Bitmap、GEO等特殊结构

**高可用性**：

- 主从复制
- 哨兵模式
- 集群模式

这些场景在实际项目中经常遇到，掌握Redis的这些应用场景对于Java开发工程师来说是非常重要的。你在项目中主要用Redis解决什么问题呢？