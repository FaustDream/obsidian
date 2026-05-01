# 有个场景，需要异步去执行操作，如何将threadLocal中的用户信息同步到异步的线程中去

在异步执行操作时，ThreadLocal变量无法自动传递到新的线程中，这是因为ThreadLocal是线程隔离的。以下是几种将ThreadLocal中的用户信息同步到异步线程的解决方案：

## 1. 手动传递方式

```java
public class UserContext {
    private static final ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userThreadLocal.set(user);
    }
    
    public static User getUser() {
        return userThreadLocal.get();
    }
    
    public static void clear() {
        userThreadLocal.remove();
    }
}

// 使用示例
@Service
public class UserService {
    
    @Async
    public void processUserData() {
        // 在主线程中获取用户信息
        User currentUser = UserContext.getUser();
        
        // 手动传递到异步线程
        CompletableFuture.runAsync(() -> {
            try {
                // 在异步线程中设置用户信息
                UserContext.setUser(currentUser);
                // 执行业务逻辑
                doSomething();
            } finally {
                // 清理ThreadLocal
                UserContext.clear();
            }
        });
    }
}
```

## 2. 使用TransmittableThreadLocal（推荐）

TransmittableThreadLocal是阿里巴巴开源的库，可以自动在异步线程间传递ThreadLocal变量。

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>transmittable-thread-local</artifactId>
    <version>2.14.2</version>
</dependency>
```

```java
public class UserContext {
    // 使用TransmittableThreadLocal替代ThreadLocal
    private static final TransmittableThreadLocal<User> userTTL = new TransmittableThreadLocal<>();
    
    public static void setUser(User user) {
        userTTL.set(user);
    }
    
    public static User getUser() {
        return userTTL.get();
    }
    
    public static void clear() {
        userTTL.remove();
    }
}

// 配置线程池装饰器
@Configuration
public class AsyncConfig {
    
    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");
        
        // 使用TtlExecutors装饰线程池
        return TtlExecutors.getTtlExecutor(executor);
    }
}
```

## 3. 自定义异步执行器

```java
@Component
public class ContextAwareAsyncExecutor {
    
    private final TaskExecutor taskExecutor;
    
    public ContextAwareAsyncExecutor(TaskExecutor taskExecutor) {
        this.taskExecutor = taskExecutor;
    }
    
    public void execute(Runnable task) {
        // 获取当前线程的用户信息
        User currentUser = UserContext.getUser();
        
        taskExecutor.execute(() -> {
            try {
                // 在异步线程中设置用户信息
                UserContext.setUser(currentUser);
                task.run();
            } finally {
                UserContext.clear();
            }
        });
    }
    
    public <T> CompletableFuture<T> submit(Callable<T> task) {
        User currentUser = UserContext.getUser();
        
        return CompletableFuture.supplyAsync(() -> {
            try {
                UserContext.setUser(currentUser);
                return task.call();
            } catch (Exception e) {
                throw new RuntimeException(e);
            } finally {
                UserContext.clear();
            }
        }, taskExecutor);
    }
}
```

## 4. 使用Spring Security的SecurityContextHolder

如果用户信息存储在Spring Security的SecurityContext中：

```java
@Service
public class SecurityContextService {
    
    @Async
    public void asyncMethod() {
        // 在主线程中获取SecurityContext
        SecurityContext context = SecurityContextHolder.getContext();
        
        CompletableFuture.runAsync(() -> {
            try {
                // 在异步线程中设置SecurityContext
                SecurityContextHolder.setContext(context);
                // 执行业务逻辑
                Authentication auth = SecurityContextHolder.getContext().getAuthentication();
                String username = auth.getName();
                // ...
            } finally {
                SecurityContextHolder.clearContext();
            }
        });
    }
}
```

## 5. 使用MDC（Mapped Diagnostic Context）

如果用户信息存储在日志的MDC中：

```java
public class MdcAwareAsyncExecutor {
    
    public void execute(Runnable task) {
        // 获取当前线程的MDC
        Map<String, String> mdcMap = MDC.getCopyOfContextMap();
        
        CompletableFuture.runAsync(() -> {
            try {
                // 在异步线程中设置MDC
                if (mdcMap != null) {
                    MDC.setContextMap(mdcMap);
                }
                task.run();
            } finally {
                MDC.clear();
            }
        });
    }
}
```

## 最佳实践建议

1. **使用TransmittableThreadLocal**：这是最简单且功能最完整的解决方案，支持线程池、ForkJoinPool等各种场景。
    
2. **注意内存泄漏**：无论使用哪种方案，都要确保在异步任务完成后清理ThreadLocal变量。
    
3. **统一封装**：建议封装一个统一的异步执行器，避免在每个地方都手动处理上下文传递。
    
4. **性能考虑**：TransmittableThreadLocal会有一定的性能开销，在高并发场景下需要权衡。
    
5. **测试验证**：在复杂的异步场景中，要充分测试上下文传递的正确性。
    

这些方案可以根据具体的业务场景和技术栈选择合适的实现方式。