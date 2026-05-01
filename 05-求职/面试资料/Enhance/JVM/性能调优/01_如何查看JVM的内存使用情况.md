# 如何查看JVM的内存使用情况

查看JVM内存使用情况有多种方法，我按照实用性和常见程度来介绍：

## 1. 命令行工具

### jstat (最常用)

```bash
# 查看垃圾回收和内存使用情况
jstat -gc [pid] 1s 10  # 每秒输出一次，共10次

# 查看堆内存使用情况
jstat -gccapacity [pid]

# 查看新生代内存使用
jstat -gcnew [pid]
```

### jmap

```bash
# 查看堆内存使用情况
jmap -heap [pid]

# 生成堆转储文件
jmap -dump:format=b,file=heap.hprof [pid]
```

### jinfo

```bash
# 查看JVM参数
jinfo -flags [pid]
```

## 2. 图形化工具

### JVisualVM

- Oracle官方提供的可视化工具
- 可以实时监控内存、CPU、线程等
- 支持内存快照分析

### JConsole

- JDK自带的监控工具
- 提供内存、线程、类加载等信息
- 支持MBean操作

## 3. 程序内部监控

### 使用Runtime类

```java
Runtime runtime = Runtime.getRuntime();
long totalMemory = runtime.totalMemory();     // 总内存
long freeMemory = runtime.freeMemory();       // 空闲内存
long maxMemory = runtime.maxMemory();         // 最大内存
long usedMemory = totalMemory - freeMemory;   // 已使用内存

System.out.println("总内存: " + totalMemory / 1024 / 1024 + "MB");
System.out.println("已用内存: " + usedMemory / 1024 / 1024 + "MB");
System.out.println("空闲内存: " + freeMemory / 1024 / 1024 + "MB");
System.out.println("最大内存: " + maxMemory / 1024 / 1024 + "MB");
```

### 使用MemoryMXBean

```java
MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();

System.out.println("堆内存使用: " + heapUsage.getUsed() / 1024 / 1024 + "MB");
System.out.println("非堆内存使用: " + nonHeapUsage.getUsed() / 1024 / 1024 + "MB");
```

## 4. 第三方监控工具

### Arthas (阿里巴巴)

```bash
# 查看JVM信息
jvm

# 查看内存使用情况
memory
```

### Spring Boot Actuator

```java
# 在application.properties中启用
management.endpoints.web.exposure.include=health,info,metrics

# 访问端点
GET /actuator/metrics/jvm.memory.used
GET /actuator/metrics/jvm.memory.max
```

## 5. 监控关键指标

### 堆内存区域

- **年轻代 (Young Generation)**：Eden区 + Survivor区
- **老年代 (Old Generation)**：长期存活对象
- **元空间 (Metaspace)**：类元数据信息

### 关键指标

- **内存使用率**：used / max * 100%
- **GC频率**：垃圾回收次数和时间
- **内存泄漏**：内存使用量持续增长

## 6. 实际应用建议

### 生产环境监控

```bash
# 设置JVM参数启用监控
-XX:+PrintGC 
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log
```

### 内存调优参数

```bash
-Xms2g          # 初始堆大小
-Xmx4g          # 最大堆大小
-XX:NewRatio=3  # 新生代与老年代比例
-XX:MaxMetaspaceSize=256m  # 元空间最大大小
```

## 扩展知识

### 内存溢出排查

1. 使用jmap生成堆转储
2. 用MAT或JVisualVM分析
3. 查找内存泄漏点
4. 优化代码或调整JVM参数

### 性能优化建议

- 合理设置堆大小，避免过大或过小
- 根据应用特点选择合适的垃圾收集器
- 监控GC日志，优化GC性能
- 定期进行内存分析，防止内存泄漏

这些方法可以帮助你全面了解JVM内存使用情况，从而进行有效的性能调优。