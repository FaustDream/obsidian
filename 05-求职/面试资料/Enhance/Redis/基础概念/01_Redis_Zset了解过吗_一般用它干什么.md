# Redis Zset了解过吗，一般用它干什么

Redis的Zset（Sorted Set，有序集合）是Redis中一个非常重要的数据结构，我来详细介绍一下。

## 什么是Zset

Zset是Redis中的有序集合，它结合了Set和Hash的特点：

- 像Set一样，元素不重复
- 每个元素都关联一个分数（score），根据分数自动排序
- 支持按分数范围查询和排名操作

## 底层实现原理

Zset的底层实现比较复杂，根据数据量采用不同策略：

**小数据量时**：使用ziplist（压缩列表）

- 当元素数量少于128个且每个元素长度小于64字节时
- 内存占用小，但插入删除相对较慢

**大数据量时**：使用skiplist（跳跃表）+ hash table

- 跳跃表保证有序性，支持范围查询
- 哈希表保证O(1)的成员查找效率

## 常见应用场景

### 1. 排行榜系统

```java
// 游戏积分排行榜
jedis.zadd("game_rank", 1000, "player1");
jedis.zadd("game_rank", 1500, "player2");

// 获取前10名
Set<String> topPlayers = jedis.zrevrange("game_rank", 0, 9);

// 获取某玩家排名
Long rank = jedis.zrevrank("game_rank", "player1");
```

### 2. 延时队列

```java
// 延时任务队列，score为执行时间戳
long executeTime = System.currentTimeMillis() + 60000; // 1分钟后执行
jedis.zadd("delay_queue", executeTime, "task_1");

// 获取到期任务
Set<String> expiredTasks = jedis.zrangeByScore("delay_queue", 0, System.currentTimeMillis());
```

### 3. 时间线/时序数据

```java
// 微博时间线，score为发布时间
jedis.zadd("user_timeline", timestamp, "weibo_id_123");

// 获取最新的微博
Set<String> recentWeibos = jedis.zrevrange("user_timeline", 0, 19);
```

### 4. 热度统计

```java
// 文章热度排序，score为点击数
jedis.zincrby("article_hot", 1, "article_123"); // 增加点击数

// 获取热门文章
Set<String> hotArticles = jedis.zrevrange("article_hot", 0, 9);
```

### 5. 地理位置应用

```java
// 基于距离的附近商店（虽然Redis有GEO类型，但也可用Zset实现）
// score可以是距离或经纬度编码
jedis.zadd("nearby_shops", distance, "shop_id");
```

## 主要命令

**基础操作**：

- `ZADD key score member` - 添加元素
- `ZREM key member` - 删除元素
- `ZCARD key` - 获取元素数量
- `ZSCORE key member` - 获取元素分数

**范围查询**：

- `ZRANGE key start stop` - 按排名范围查询（升序）
- `ZREVRANGE key start stop` - 按排名范围查询（降序）
- `ZRANGEBYSCORE key min max` - 按分数范围查询

**排名操作**：

- `ZRANK key member` - 获取元素排名（升序）
- `ZREVRANK key member` - 获取元素排名（降序）

## 性能特点

- 插入、删除、查找：O(log N)
- 范围查询：O(log N + M)，M为返回元素数量
- 获取排名：O(log N)

## 注意事项

1. **内存使用**：Zset比普通Set占用更多内存，因为需要存储score
2. **分数精度**：score是double类型，可能存在精度问题
3. **相同分数**：相同分数的元素按字典序排序
4. **数据一致性**：在集群环境下要注意数据分片和一致性问题

## 扩展知识

**与其他数据结构的区别**：

- List：有序但允许重复，不支持按值快速查找
- Set：无序且不重复，不支持排序
- Hash：键值对存储，不支持排序

**实际项目考虑**：

- 选择合适的分数设计（时间戳、权重值等）
- 定期清理过期数据避免内存膨胀
- 在高并发场景下考虑使用Lua脚本保证原子性

Redis Zset是一个功能强大的数据结构，特别适合需要排序和排名的业务场景，在互联网应用中使用非常广泛。