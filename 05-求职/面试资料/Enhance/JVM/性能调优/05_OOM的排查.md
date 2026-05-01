# OOM的排查

## OOM排查方法与实践

### 1. 什么是OOM

OOM（Out Of Memory）是Java应用程序中最常见的内存问题之一，当JVM无法分配足够的内存来满足应用程序需求时就会抛出OutOfMemoryError。

### 2. 常见的OOM类型

**Java heap space**

- 堆内存不足，最常见的OOM类型
- 通常由内存泄漏或堆内存配置过小导致

**PermGen space / Metaspace**

- 永久代（JDK8之前）或元空间（JDK8+）内存不足
- 常见于类加载过多的场景

**GC overhead limit exceeded**

- GC耗时过长，超过98%的时间用于GC但回收内存少于2%

**Direct buffer memory**

- 直接内存不足，通常与NIO相关

### 3. OOM排查步骤

#### 3.1 收集基础信息

```bash
# 查看JVM参数
jps -v

# 查看堆内存使用情况
jstat -gc <pid> 1s

# 查看内存区域详情
jmap -heap <pid>
```

#### 3.2 生成堆转储文件

```bash
# 手动生成heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# 自动生成（JVM参数）
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dump
```

#### 3.3 分析堆转储文件

使用专业工具分析：

- **Eclipse MAT**：最常用的内存分析工具
- **JVisualVM**：JDK自带的可视化工具
- **JProfiler**：商业化性能分析工具

### 4. 具体排查技巧

#### 4.1 使用MAT分析

```java
// 典型的内存泄漏代码示例
public class MemoryLeakExample {
    private static List<Object> list = new ArrayList<>();
    
    public void addObjects() {
        for (int i = 0; i < 1000000; i++) {
            list.add(new Object()); // 对象无法被回收
        }
    }
}
```

MAT分析要点：

- 查看Histogram找到占用内存最多的对象
- 使用Dominator Tree分析对象引用关系
- 通过Leak Suspects自动识别可能的内存泄漏

#### 4.2 实时监控分析

```bash
# 实时监控GC情况
jstat -gc <pid> 1s 10

# 监控类加载情况
jstat -class <pid>

# 查看线程堆栈
jstack <pid>
```

### 5. 常见OOM场景及解决方案

#### 5.1 内存泄漏场景

```java
// 监听器未移除
public class ListenerLeak {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);
        // 忘记提供remove方法或在适当时机调用
    }
}

// 线程池任务积压
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10, 10, 0L, TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<>() // 无界队列可能导致OOM
);
```

#### 5.2 大对象处理

```java
// 避免一次性加载大量数据
public List<User> getAllUsers() {
    // 错误做法：一次性加载所有用户
    return userRepository.findAll();
    
    // 正确做法：分页处理
    // return userRepository.findAll(PageRequest.of(0, 1000));
}
```

### 6. 预防措施

#### 6.1 JVM参数优化

```bash
# 设置合适的堆内存大小
-Xms4g -Xmx4g

# 启用GC日志
-XX:+UseG1GC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps

# 内存监控参数
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/app/dumps/
```

#### 6.2 代码层面预防

```java
// 使用try-with-resources确保资源释放
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // 处理文件
} catch (IOException e) {
    // 异常处理
}

// 合理使用缓存
@Service
public class UserService {
    @Autowired
    private CacheManager cacheManager;
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id);
    }
}
```

### 7. 监控告警体系

#### 7.1 应用级监控

```java
// 使用Micrometer进行内存监控
@Component
public class MemoryMetrics {
    private final MeterRegistry meterRegistry;
    
    public MemoryMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        Gauge.builder("jvm.memory.used")
            .register(meterRegistry, this, MemoryMetrics::getUsedMemory);
    }
    
    private double getUsedMemory() {
        return Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory();
    }
}
```

#### 7.2 告警配置

- 设置堆内存使用率超过80%时告警
- 配置GC时间过长告警
- 监控Full GC频率异常

### 8. 扩展知识

#### 8.1 不同垃圾收集器的影响

- **Serial GC**：适合小型应用
- **Parallel GC**：适合吞吐量优先的应用
- **G1 GC**：适合低延迟要求的大堆应用
- **ZGC/Shenandoah**：超低延迟垃圾收集器

#### 8.2 容器化环境的特殊考虑

```bash
# Docker容器内存限制
docker run -m 4g myapp

# JVM自动感知容器限制（JDK8u131+）
-XX:+UnlockExperimentalVMOptions
-XX:+UseCGroupMemoryLimitForHeap
```

通过系统性的排查方法和预防措施，可以有效识别和解决OOM问题，确保Java应用的稳定运行。