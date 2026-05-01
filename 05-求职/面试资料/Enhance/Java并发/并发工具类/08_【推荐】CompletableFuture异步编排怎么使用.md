# CompletableFuture异步编排怎么使用

CompletableFuture是Java 8引入的强大异步编程工具，它提供了丰富的API来处理异步任务的编排和组合。

## 基本用法

### 1. 创建CompletableFuture
> supplyAsync() 有返回值，runAsync() 无返回值

```java
// 创建已完成的CompletableFuture
CompletableFuture<String> future1 = CompletableFuture.completedFuture("Hello");

// 异步执行任务
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    // 模拟耗时操作
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    return "World";
});

// 异步执行无返回值任务
CompletableFuture<Void> future3 = CompletableFuture.runAsync(() -> {
    System.out.println("执行异步任务");
});
```

### 2. 链式调用和转换

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")           // 转换结果
    .thenApply(String::toUpperCase)         // 继续转换
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "!"));  // 组合另一个Future
```

### 3. 处理结果和异常

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        throw new RuntimeException("出错了");
    }
    return "成功";
})
.handle((result, throwable) -> {
    if (throwable != null) {
        return "处理异常: " + throwable.getMessage();
    }
    return result;
})
.whenComplete((result, throwable) -> {
    System.out.println("任务完成: " + result);
});
```

## 高级异步编排

### 1. 多任务并行执行

```java
public class AsyncService {
    
    public CompletableFuture<String> getUserInfo(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // 模拟查询用户信息
            return "用户信息-" + userId;
        });
    }
    
    public CompletableFuture<String> getOrderInfo(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // 模拟查询订单信息
            return "订单信息-" + userId;
        });
    }
    
    public CompletableFuture<String> getAccountInfo(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // 模拟查询账户信息
            return "账户信息-" + userId;
        });
    }
    
    // 并行获取所有信息
    public CompletableFuture<Map<String, String>> getAllUserData(Long userId) {
        CompletableFuture<String> userFuture = getUserInfo(userId);
        CompletableFuture<String> orderFuture = getOrderInfo(userId);
        CompletableFuture<String> accountFuture = getAccountInfo(userId);
        
        return CompletableFuture.allOf(userFuture, orderFuture, accountFuture)
            .thenApply(v -> {
                Map<String, String> result = new HashMap<>();
                result.put("user", userFuture.join());
                result.put("order", orderFuture.join());
                result.put("account", accountFuture.join());
                return result;
            });
    }
}
```

### 2. 任务依赖和组合

```java
public class TaskOrchestration {
    
    public CompletableFuture<String> processOrder(String orderId) {
        return CompletableFuture.supplyAsync(() -> {
            // 步骤1: 验证订单
            System.out.println("验证订单: " + orderId);
            return orderId;
        })
        .thenCompose(this::checkInventory)      // 步骤2: 检查库存
        .thenCompose(this::calculatePrice)     // 步骤3: 计算价格
        .thenCompose(this::processPayment)     // 步骤4: 处理支付
        .thenApply(result -> {
            System.out.println("订单处理完成: " + result);
            return result;
        });
    }
    
    private CompletableFuture<String> checkInventory(String orderId) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("检查库存: " + orderId);
            return orderId + "-库存充足";
        });
    }
    
    private CompletableFuture<String> calculatePrice(String result) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("计算价格: " + result);
            return result + "-价格100元";
        });
    }
    
    private CompletableFuture<String> processPayment(String result) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("处理支付: " + result);
            return result + "-支付成功";
        });
    }
}
```

### 3. 超时和异常处理

```java
public class TimeoutExample {
    
    public CompletableFuture<String> processWithTimeout(String data) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                // 模拟耗时操作
                Thread.sleep(5000);
                return "处理完成: " + data;
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException("任务被中断", e);
            }
        })
        .orTimeout(3, TimeUnit.SECONDS)  // 设置超时时间
        .exceptionally(throwable -> {
            if (throwable instanceof TimeoutException) {
                return "任务超时，返回默认值";
            }
            return "处理异常: " + throwable.getMessage();
        });
    }
}
```

## 实际应用场景

### 1. 微服务调用编排

```java
@Service
public class OrderService {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @Autowired
    private InventoryServiceClient inventoryServiceClient;
    
    @Autowired
    private PaymentServiceClient paymentServiceClient;
    
    public CompletableFuture<OrderResult> createOrder(CreateOrderRequest request) {
        // 并行调用多个服务
        CompletableFuture<User> userFuture = CompletableFuture
            .supplyAsync(() -> userServiceClient.getUser(request.getUserId()));
            
        CompletableFuture<Boolean> inventoryFuture = CompletableFuture
            .supplyAsync(() -> inventoryServiceClient.checkStock(request.getProductId()));
            
        // 等待前置条件完成后再调用支付服务
        return CompletableFuture.allOf(userFuture, inventoryFuture)
            .thenCompose(v -> {
                User user = userFuture.join();
                Boolean hasStock = inventoryFuture.join();
                
                if (!hasStock) {
                    return CompletableFuture.completedFuture(
                        OrderResult.failed("库存不足"));
                }
                
                // 调用支付服务
                return CompletableFuture.supplyAsync(() -> 
                    paymentServiceClient.processPayment(user.getId(), request.getAmount()));
            })
            .thenApply(paymentResult -> {
                if (paymentResult.isSuccess()) {
                    return OrderResult.success("订单创建成功");
                } else {
                    return OrderResult.failed("支付失败");
                }
            });
    }
}
```

### 2. 自定义线程池

```java
@Configuration
public class AsyncConfig {
    
    @Bean("asyncExecutor")
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class AsyncTaskService {
    
    @Autowired
    @Qualifier("asyncExecutor")
    private Executor executor;
    
    public CompletableFuture<String> processTask(String data) {
        return CompletableFuture.supplyAsync(() -> {
            // 处理任务
            return "处理结果: " + data;
        }, executor);
    }
}
```

## 关键知识点

### 1. 常用方法对比

- **thenApply**: 对结果进行转换
- **thenCompose**: 组合另一个CompletableFuture
- **thenCombine**: 组合两个CompletableFuture的结果
- **allOf**: 等待所有任务完成
- **anyOf**: 等待任何一个任务完成

### 2. 异常处理方式

- **exceptionally**: 处理异常并返回默认值
- **handle**: 同时处理正常结果和异常
- **whenComplete**: 任务完成时执行（类似finally）

### 3. 性能优化建议

- 使用自定义线程池避免默认ForkJoinPool的限制
- 合理设置超时时间防止任务无限等待
- 注意避免过度创建CompletableFuture导致内存问题
- 在高并发场景下监控线程池状态

CompletableFuture的异步编排能力让我们能够优雅地处理复杂的异步任务组合，提高系统的响应性能和用户体验。