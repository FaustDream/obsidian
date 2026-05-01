# 频繁的FullGC是什么情况，怎么解决

频繁的Full GC是Java应用性能问题中的一个严重信号，我来详细解释一下这个问题和解决方案。

## 什么是频繁的Full GC

Full GC是垃圾回收器对整个堆内存（年轻代+老年代）进行的完整清理，它会暂停所有应用线程（Stop The World）。频繁的Full GC通常指：

- Full GC间隔时间很短（比如几秒钟就执行一次）
- 每次Full GC耗时很长（比如超过1秒）
- Full GC后内存回收效果不明显

## 造成频繁Full GC的常见原因

**1. 内存泄漏**

```java
// 典型的内存泄漏例子
public class MemoryLeakExample {
    private static List<Object> cache = new ArrayList<>();
    
    public void addToCache(Object obj) {
        cache.add(obj); // 对象永远不会被清理
    }
}
```

**2. 老年代空间不足**

- 大对象直接进入老年代
- 年轻代对象过早晋升到老年代
- 老年代空间设置过小

**3. 元空间（Metaspace）溢出**

```java
// 动态生成大量类导致元空间溢出
public class MetaspaceExample {
    public void generateClasses() {
        // 使用字节码生成框架如ASM、CGLib动态创建类
        // 如果不注意清理，会导致元空间溢出
    }
}
```

**4. 不合理的GC参数配置**

## 解决方案

### 1. 分析和监控

**使用GC日志分析**

```bash
# 开启GC日志
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log

# Java 9+使用统一日志
-Xlog:gc*:gc.log:time
```

**内存分析工具**

- jstat: 实时监控GC情况
- jmap: 生成堆转储文件
- MAT/Eclipse Memory Analyzer: 分析内存泄漏

### 2. 代码层面优化

**避免内存泄漏**

```java
// 正确的缓存使用方式
public class CacheExample {
    // 使用WeakReference或有界缓存
    private final Map<String, WeakReference<Object>> cache = 
        new ConcurrentHashMap<>();
    
    // 或使用Guava Cache等成熟的缓存框架
    private final Cache<String, Object> cache = 
        CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
}
```

**减少大对象创建**

```java
// 避免在循环中创建大对象
public class ObjectCreationExample {
    // 错误方式
    public void badExample() {
        for (int i = 0; i < 1000; i++) {
            byte[] largeArray = new byte[1024 * 1024]; // 每次创建1MB
        }
    }
    
    // 正确方式
    private static final byte[] REUSABLE_BUFFER = new byte[1024 * 1024];
    public void goodExample() {
        // 复用缓冲区
        Arrays.fill(REUSABLE_BUFFER, (byte) 0);
    }
}
```

### 3. JVM参数调优

**堆内存调优**

```bash
# 增加堆内存大小
-Xms4g -Xmx4g

# 调整新生代和老年代比例
-XX:NewRatio=2  # 老年代:新生代 = 2:1
-XX:SurvivorRatio=8  # Eden:Survivor = 8:1

# 控制对象晋升到老年代的阈值
-XX:MaxTenuringThreshold=15
```

**选择合适的垃圾收集器**

```bash
# G1GC (推荐用于大堆内存)
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200

# ZGC (超低延迟收集器)
-XX:+UseZGC

# Parallel GC (吞吐量优先)
-XX:+UseParallelGC
```

### 4. 应用架构优化

**对象池化**

```java
// 使用对象池减少GC压力
public class ObjectPoolExample {
    private final Queue<StringBuilder> stringBuilderPool = 
        new ConcurrentLinkedQueue<>();
    
    public StringBuilder borrowStringBuilder() {
        StringBuilder sb = stringBuilderPool.poll();
        return sb != null ? sb : new StringBuilder();
    }
    
    public void returnStringBuilder(StringBuilder sb) {
        sb.setLength(0); // 重置状态
        stringBuilderPool.offer(sb);
    }
}
```

**分批处理**

```java
// 分批处理大量数据，避免一次性加载
public class BatchProcessExample {
    public void processLargeDataset(List<Data> dataset) {
        int batchSize = 1000;
        for (int i = 0; i < dataset.size(); i += batchSize) {
            List<Data> batch = dataset.subList(i, 
                Math.min(i + batchSize, dataset.size()));
            processBatch(batch);
            // 显式建议GC（谨慎使用）
            System.gc();
        }
    }
}
```

## 扩展知识点

### 不同垃圾收集器的特点

|收集器|特点|适用场景|
|---|---|---|
|Serial GC|单线程，暂停时间长|小型应用|
|Parallel GC|多线程，吞吐量高|批处理应用|
|CMS|并发收集，低延迟|响应敏感应用|
|G1GC|可预测暂停时间|大堆内存应用|
|ZGC/Shenandoah|超低延迟|极致响应要求|

### 监控指标

**关键GC指标**

- GC频率：每分钟GC次数
- GC耗时：平均每次GC时间
- 内存回收率：GC后内存释放比例
- 应用暂停时间：STW时间占比

通过系统性的分析和优化，可以有效解决频繁Full GC的问题，显著提升应用性能。