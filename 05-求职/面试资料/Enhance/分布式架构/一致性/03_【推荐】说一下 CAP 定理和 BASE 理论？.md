CAP定理(一致性、可用性、分区容错性)
1. 分布式系统节点通过网络连接，一定会出现分区问题(P)
2. 当分区出现时，系统的一致性（C）和可用性（A）就无法同时满足，只能实现CP或AP

BASE理论是对CAP的一种解决思路
1. 基本可用（Basically Available）：布式系统在出现故障时，允许损失部分可用性，即保证核心可用。
2. 软状态（SoftState）：一定时间内，允许出现中间状态，比如临时的不一致状态。
3. 最终一致解决（EventuallyConsistent）：虽然无法保证强一致性，但是在软状态结束后，最终达到数据一致。

-----
-----

## CAP定理

CAP定理是分布式系统设计中的重要理论，由Eric Brewer在2000年提出。CAP代表三个特性：

### 三个核心特性

**一致性（Consistency）**

- 所有节点在同一时刻看到相同的数据
- 当数据更新后，所有节点都能立即读到最新值
- 类似于数据库的ACID中的一致性

**可用性（Availability）**

- 系统在任何时候都能响应用户请求
- 即使部分节点故障，系统仍能正常提供服务
- 通常用响应时间和正常运行时间来衡量

**分区容错性（Partition Tolerance）**

- 系统能够容忍网络分区故障
- 即使节点间通信中断，系统仍能继续运行
- 这是分布式系统必须面对的现实问题

### CAP定理核心观点

**在分布式系统中，最多只能同时保证CAP中的两个特性，无法三者兼得。**

### 实际应用场景

- **CP系统**：如MongoDB、HBase，保证一致性和分区容错，但可能牺牲可用性
- **AP系统**：如Cassandra、DynamoDB，保证可用性和分区容错，但最终一致性
- **CA系统**：理论存在，但在真实分布式环境中很难实现

## BASE理论

BASE理论是对CAP定理的延伸，由eBay架构师Dan Pritchett提出，是对传统ACID特性的补充。

### 三个核心概念

**基本可用（Basically Available）**

- 系统能够基本运行，允许损失部分可用性
- 比如响应时间延长、功能降级等
- 例：双11期间，部分非核心功能可能暂停

**软状态（Soft State）**

- 系统状态可以有一定时间的不一致
- 数据不要求任何时刻都完全一致
- 允许中间状态的存在

**最终一致性（Eventually Consistent）**

- 经过一段时间后，系统会达到一致状态
- 不保证实时一致性，但保证最终一致性
- 这是一种较弱的一致性要求

### BASE与ACID的对比

|特性|ACID|BASE|
|---|---|---|
|一致性|强一致性|最终一致性|
|可用性|一般|高可用性|
|性能|相对较低|高性能|
|复杂度|相对简单|复杂|

## 实际应用示例

### 电商系统中的应用

```java
// 订单系统示例 - BASE理论应用
public class OrderService {
    
    // 1. 基本可用：即使库存服务暂时不可用，订单仍可创建
    public Order createOrder(OrderRequest request) {
        Order order = new Order();
        order.setStatus("PENDING");
        
        // 异步处理库存检查
        asyncCheckInventory(order);
        
        return orderRepository.save(order);
    }
    
    // 2. 软状态：订单在处理过程中可能处于中间状态
    private void asyncCheckInventory(Order order) {
        CompletableFuture.runAsync(() -> {
            if (inventoryService.checkAndReserve(order)) {
                order.setStatus("CONFIRMED");
            } else {
                order.setStatus("CANCELLED");
            }
            // 3. 最终一致性：最终订单状态会确定
            orderRepository.save(order);
        });
    }
}
```

## 相关知识扩展

### 一致性模型层次

1. **强一致性**：如线性一致性、顺序一致性
2. **弱一致性**：如因果一致性、最终一致性
3. **无一致性**：完全异步，无任何保证

### 分布式系统中的权衡

- **延迟 vs 一致性**：强一致性通常意味着更高延迟
- **可用性 vs 一致性**：高可用性可能需要牺牲一致性要求
- **成本 vs 性能**：更好的性能通常需要更高的成本

### 现代分布式系统实践

现代系统往往采用混合策略：

- 核心业务数据采用强一致性（CP）
- 非核心数据采用最终一致性（AP）
- 根据业务场景灵活选择一致性级别

这两个理论为分布式系统设计提供了重要的理论指导，帮助我们在实际项目中做出合适的技术选择。