# 如果CPU占用过高，发生OOM，如何排查

当遇到CPU占用过高和OOM（Out of Memory）问题时，需要系统性地进行排查。我来详细介绍排查思路和具体方法：

## CPU占用过高排查

### 1. 系统层面监控

```bash
# 查看系统整体CPU使用情况
top
htop

# 查看具体进程的CPU占用
ps aux --sort=-%cpu | head -10

# 实时监控CPU使用率
vmstat 1
```

### 2. Java应用层面排查

```bash
# 找到Java进程PID
jps -l

# 查看线程级别的CPU占用
top -H -p <pid>

# 将占用CPU最高的线程ID转换为16进制
printf "%x\n" <thread_id>

# 生成线程dump分析
jstack <pid> | grep -A 10 <hex_thread_id>
```

### 3. 常见CPU占用过高原因

- 死循环或无限循环
- 频繁的GC（垃圾回收）
- 线程竞争激烈
- 大量正则表达式匹配
- 序列化/反序列化操作过多

## OOM排查方法

### 1. 开启JVM参数记录OOM信息

```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dump
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
```

### 2. 内存分析工具

```bash
# 生成堆转储文件
jmap -dump:format=b,file=heap.hprof <pid>

# 查看堆内存使用情况
jmap -heap <pid>

# 查看对象统计信息
jmap -histo <pid>
```

### 3. 使用专业工具分析

- **Eclipse MAT**：分析heap dump文件
- **JProfiler**：实时监控内存使用
- **VisualVM**：JVM自带的监控工具

## 实际排查案例

### 案例1：循环引用导致的内存泄漏

```java
// 问题代码示例
public class MemoryLeakExample {
    private List<Object> cache = new ArrayList<>();
    
    public void addToCache(Object obj) {
        cache.add(obj); // 只添加不清理
    }
}

// 解决方案
public class FixedExample {
    private Map<String, Object> cache = new ConcurrentHashMap<>();
    
    public void addToCache(String key, Object obj) {
        // 设置缓存大小限制
        if (cache.size() > 1000) {
            cache.clear();
        }
        cache.put(key, obj);
    }
}
```

### 案例2：线程池配置不当

```java
// 问题配置
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    50, 200, 60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>() // 无界队列可能导致OOM
);

// 优化配置
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10, 50, 60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(1000), // 有界队列
    new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
```

## 预防措施

### 1. 代码层面

- 及时关闭资源（IO流、数据库连接）
- 合理使用缓存，设置过期时间
- 避免在循环中创建大量对象
- 使用对象池复用对象

### 2. JVM调优

```bash
# 堆内存设置
-Xms2g -Xmx4g

# 新生代和老年代比例
-XX:NewRatio=3

# GC算法选择
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

### 3. 监控告警

- 设置CPU和内存使用率告警
- 监控GC频率和耗时
- 关注应用响应时间

## 排查工具总结

|工具|用途|命令示例|
|---|---|---|
|jstat|GC监控|`jstat -gc <pid> 1s`|
|jmap|内存分析|`jmap -histo <pid>`|
|jstack|线程分析|`jstack <pid>`|
|jinfo|参数查看|`jinfo -flags <pid>`|

通过这套系统化的排查方法，可以有效定位和解决CPU占用过高和OOM问题。关键是要结合具体的业务场景，从系统监控、应用日志、代码审查等多个维度进行分析。