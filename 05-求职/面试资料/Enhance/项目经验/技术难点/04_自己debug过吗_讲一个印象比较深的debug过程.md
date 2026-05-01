# 自己debug过吗，讲一个印象比较深的debug过程

我来分享一个印象深刻的debug经历，这是一个典型的多线程并发问题。

## 问题背景

这是在一个电商系统中，用户反馈偶尔会出现订单状态不一致的问题 - 明明支付成功了，但订单状态还是"待支付"，而且这个问题很难复现，只在高并发时偶发。

## 问题现象

```java
// 简化的订单处理代码
public class OrderService {
    private Map<String, Order> orderCache = new HashMap<>();
    
    public void processPayment(String orderId, PaymentResult result) {
        Order order = orderCache.get(orderId);
        if (order != null && result.isSuccess()) {
            order.setStatus("PAID");
            orderCache.put(orderId, order);
            // 发送通知
            notifyUser(order);
        }
    }
}
```

## Debug过程

**第一步：日志分析**

- 发现同一个订单ID在短时间内有多次支付处理记录
- 但最终状态却是错误的

**第二步：怀疑并发问题**

- 添加详细日志，记录每次操作的线程ID
- 发现确实有多个线程同时处理同一订单

**第三步：复现问题**

- 写了一个多线程测试用例

```java
@Test
public void testConcurrentPayment() throws InterruptedException {
    OrderService service = new OrderService();
    String orderId = "12345";
    
    // 创建100个线程同时处理支付
    CountDownLatch latch = new CountDownLatch(100);
    for (int i = 0; i < 100; i++) {
        new Thread(() -> {
            service.processPayment(orderId, new PaymentResult(true));
            latch.countDown();
        }).start();
    }
    latch.await();
}
```

**第四步：定位根本原因** 问题出在HashMap不是线程安全的，在高并发情况下：

1. 多个线程同时执行`get()`操作
2. 同时获取到同一个Order对象的引用
3. 都对这个对象进行修改
4. 最后执行`put()`时，可能出现数据覆盖或丢失

## 解决方案

**方案一：使用ConcurrentHashMap + synchronized**

```java
public class OrderService {
    private ConcurrentHashMap<String, Order> orderCache = new ConcurrentHashMap<>();
    
    public synchronized void processPayment(String orderId, PaymentResult result) {
        Order order = orderCache.get(orderId);
        if (order != null && result.isSuccess()) {
            order.setStatus("PAID");
            orderCache.put(orderId, order);
            notifyUser(order);
        }
    }
}
```

**方案二：使用分布式锁（最终采用）**

```java
public class OrderService {
    private RedisTemplate redisTemplate;
    
    public void processPayment(String orderId, PaymentResult result) {
        String lockKey = "order_lock:" + orderId;
        Boolean lockAcquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, "1", 30, TimeUnit.SECONDS);
            
        if (lockAcquired) {
            try {
                // 处理支付逻辑
                Order order = orderCache.get(orderId);
                if (order != null && result.isSuccess()) {
                    order.setStatus("PAID");
                    orderCache.put(orderId, order);
                    notifyUser(order);
                }
            } finally {
                redisTemplate.delete(lockKey);
            }
        }
    }
}
```

## 经验总结

1. **并发问题往往在高负载时才暴露**，平时测试很难发现
2. **日志是debug的最好朋友**，关键操作要记录线程信息
3. **HashMap在多线程环境下不仅不安全，还可能死循环**
4. **分布式系统中，单机锁往往不够用**，需要考虑分布式锁
5. **压力测试很重要**，能提前发现并发问题

## 扩展知识点

这个问题还涉及到：

- **线程安全的集合类**：ConcurrentHashMap、Vector等
- **分布式锁的实现**：Redis、Zookeeper、数据库
- **幂等性设计**：防止重复处理
- **事务的ACID特性**：原子性在分布式环境中的挑战

你在实际开发中遇到过类似的并发问题吗？