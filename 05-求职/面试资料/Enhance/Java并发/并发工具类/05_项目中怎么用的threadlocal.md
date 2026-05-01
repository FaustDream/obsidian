# 项目中怎么用的threadlocal

ThreadLocal在项目中的使用主要体现在以下几个场景：

## 1. 用户上下文管理

最常见的用途是在Web应用中传递用户信息，避免在每个方法间传递用户参数：

```java
public class UserContext {
    private static final ThreadLocal<User> userHolder = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userHolder.set(user);
    }
    
    public static User getCurrentUser() {
        return userHolder.get();
    }
    
    public static void clear() {
        userHolder.remove(); // 防止内存泄漏
    }
}

// 在Filter或Interceptor中设置
@WebFilter("/*")
public class UserContextFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        try {
            User user = getCurrentUserFromSession(request);
            UserContext.setUser(user);
            chain.doFilter(request, response);
        } finally {
            UserContext.clear(); // 请求结束后清理
        }
    }
}
```

## 2. 数据库连接管理

在多数据源场景中，用ThreadLocal存储当前线程应该使用的数据源：

```java
public class DataSourceContext {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
    
    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }
    
    public static String getDataSource() {
        return contextHolder.get();
    }
    
    public static void clear() {
        contextHolder.remove();
    }
}

// 在Service层根据业务逻辑切换数据源
@Service
public class OrderService {
    public void processOrder(Order order) {
        if (order.getType() == OrderType.VIP) {
            DataSourceContext.setDataSource("vip_db");
        } else {
            DataSourceContext.setDataSource("normal_db");
        }
        // 后续的DAO操作会自动使用对应的数据源
    }
}
```

## 3. 事务管理

Spring框架内部大量使用ThreadLocal来管理事务状态：

```java
// Spring的TransactionSynchronizationManager内部实现
public abstract class TransactionSynchronizationManager {
    private static final ThreadLocal<Map<Object, Object>> resources = 
        new NamedThreadLocal<>("Transactional resources");
    
    private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations = 
        new NamedThreadLocal<>("Transaction synchronizations");
}
```

## 4. 日志追踪

在微服务架构中，使用ThreadLocal存储链路追踪ID：

```java
public class TraceContext {
    private static final ThreadLocal<String> traceId = new ThreadLocal<>();
    
    public static void setTraceId(String id) {
        traceId.set(id);
        // 同时设置到MDC中，方便日志输出
        MDC.put("traceId", id);
    }
    
    public static String getTraceId() {
        return traceId.get();
    }
    
    public static void clear() {
        traceId.remove();
        MDC.clear();
    }
}
```

## 5. 性能监控

存储方法执行时间等性能指标：

```java
public class PerformanceContext {
    private static final ThreadLocal<Long> startTime = new ThreadLocal<>();
    
    public static void startTiming() {
        startTime.set(System.currentTimeMillis());
    }
    
    public static long getElapsedTime() {
        Long start = startTime.get();
        return start == null ? 0 : System.currentTimeMillis() - start;
    }
    
    public static void clear() {
        startTime.remove();
    }
}
```

## 使用注意事项

**内存泄漏防范**：

- 在Web应用中，线程池会复用线程，必须在请求结束时调用`remove()`
- 使用`ThreadLocalMap`的弱引用特性，但仍需要手动清理

**线程池环境**：

- 线程池复用线程时，ThreadLocal的值可能会"泄漏"到下一个任务
- 建议使用`TransmittableThreadLocal`来解决父子线程传递问题

**性能考虑**：

- ThreadLocal的get/set操作是O(1)，但创建过多ThreadLocal会影响性能
- 避免在高并发场景中频繁创建和销毁ThreadLocal

## 扩展知识

**InheritableThreadLocal**：可以将父线程的ThreadLocal值传递给子线程，但在线程池环境中需要特别注意。

**TransmittableThreadLocal**（阿里开源）：解决了在使用线程池等会复用线程的组件情况下，ThreadLocal值的传递问题。

ThreadLocal在项目中是一个非常实用的工具，主要用于在同一线程内传递上下文信息，避免方法参数的层层传递，但使用时一定要注意内存泄漏问题。