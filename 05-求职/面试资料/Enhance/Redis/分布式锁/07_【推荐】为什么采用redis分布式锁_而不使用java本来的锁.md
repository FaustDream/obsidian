# 为什么采用redis分布式锁，而不使用java本来的锁

Redis分布式锁相比Java原生锁的优势主要体现在分布式环境下的应用场景：

## Java原生锁的局限性

Java的`synchronized`和`ReentrantLock`等锁机制只能在**单个JVM进程内**生效。在分布式系统中，多个服务实例运行在不同的JVM中，Java原生锁无法跨进程协调，会导致：

```java
// 在分布式环境下，这种锁无法保证全局互斥
public synchronized void processOrder(String orderId) {
    // 多个服务实例可能同时执行这段代码
    // 造成重复处理订单的问题
}
```

## Redis分布式锁的优势

### 1. 跨进程/跨服务器协调

Redis作为独立的中间件，可以为多个应用实例提供统一的锁服务：

```java
public class RedisDistributedLock {
    public boolean tryLock(String lockKey, String requestId, int expireTime) {
        // SET key value NX EX expireTime
        String result = jedis.set(lockKey, requestId, "NX", "EX", expireTime);
        return "OK".equals(result);
    }
}
```

### 2. 高可用性

- Redis支持主从复制、哨兵模式、集群模式
- 即使某个Redis节点故障，锁服务仍可正常运行

### 3. 性能优越

- 基于内存操作，响应速度快
- 支持高并发访问
- 网络开销相对较小

## 典型应用场景对比

### 库存扣减场景

```java
// Java原生锁 - 只能在单个服务实例内生效
public synchronized boolean decreaseStock(String productId, int quantity) {
    // 问题：多个服务实例可能同时操作同一商品库存
    return stockService.decrease(productId, quantity);
}

// Redis分布式锁 - 全局互斥
public boolean decreaseStock(String productId, int quantity) {
    String lockKey = "stock_lock:" + productId;
    if (redisLock.tryLock(lockKey, requestId, 10)) {
        try {
            return stockService.decrease(productId, quantity);
        } finally {
            redisLock.unlock(lockKey, requestId);
        }
    }
    return false;
}
```

## 技术实现对比

### Java原生锁特点

- **作用域**：单JVM进程内
- **性能**：极高（无网络开销）
- **复杂度**：低
- **适用场景**：单机应用、线程间同步

### Redis分布式锁特点

- **作用域**：跨进程、跨服务器
- **性能**：高（有网络开销）
- **复杂度**：中等（需考虑网络异常、锁超时等）
- **适用场景**：分布式系统、微服务架构

## 扩展知识点

### 1. Redisson框架

提供了更完善的分布式锁实现，支持：

- 锁的自动续期
- 可重入特性
- 公平锁 / 非公平锁
- 读写锁

### 2. 其他分布式锁方案

- **Zookeeper**：基于临时顺序节点，强一致性
- **数据库**：基于唯一索引，简单但性能较差
- **Etcd**：云原生环境下的选择

### 3. 分布式锁的挑战

- **锁超时问题**：业务执行时间超过锁超时时间
- **网络分区**：Redis不可达时的处理策略
- **脑裂问题**：主从切换时可能出现的一致性问题

选择Redis分布式锁的核心原因是：**在分布式环境下，需要一个独立的、高性能的协调服务来实现跨进程的互斥访问控制**，这是Java原生锁无法提供的能力。