# java项目垃圾回收器是哪种

Java项目中的垃圾回收器类型主要取决于JVM版本和配置。让我详细介绍一下：

## 常见的垃圾回收器类型

### 1. **Serial GC（串行回收器）**

- **适用场景**：单核CPU或小型应用
- **特点**：单线程执行，会暂停所有用户线程
- **启用参数**：`-XX:+UseSerialGC`

### 2. **Parallel GC（并行回收器）**

- **适用场景**：多核CPU，对吞吐量要求高的应用
- **特点**：多线程并行执行垃圾回收
- **启用参数**：`-XX:+UseParallelGC`
- **注意**：JDK 8及以前版本的默认回收器

### 3. **CMS GC（并发标记清除）**

- **适用场景**：对响应时间敏感的应用
- **特点**：低延迟，但可能产生内存碎片
- **启用参数**：`-XX:+UseConcMarkSweepGC`
- **状态**：JDK 14后已废弃

### 4. **G1 GC（Garbage First）**

- **适用场景**：大内存应用（>4GB）
- **特点**：可预测的停顿时间，适合低延迟要求
- **启用参数**：`-XX:+UseG1GC`
- **注意**：JDK 9+的默认回收器

### 5. **ZGC（Z Garbage Collector）**

- **适用场景**：超大内存应用，极低延迟要求
- **特点**：停顿时间<10ms，支持TB级内存
- **启用参数**：`-XX:+UseZGC`
- **版本**：JDK 11+实验性，JDK 15+生产就绪

## 如何确定当前项目使用的垃圾回收器

### 1. **通过JVM参数查看**

```bash
# 查看JVM启动参数
jps -v
# 或者查看应用程序启动脚本
```

### 2. **通过代码获取**

```java
import java.lang.management.ManagementFactory;
import java.lang.management.GarbageCollectorMXBean;

public class GCInfo {
    public static void main(String[] args) {
        for (GarbageCollectorMXBean gcBean : 
             ManagementFactory.getGarbageCollectorMXBeans()) {
            System.out.println("GC Name: " + gcBean.getName());
            System.out.println("Collection Count: " + gcBean.getCollectionCount());
            System.out.println("Collection Time: " + gcBean.getCollectionTime() + "ms");
        }
    }
}
```

### 3. **通过监控工具**

- **JConsole**：JDK自带的监控工具
- **JVisualVM**：可视化监控工具
- **命令行**：`jstat -gc <pid>`

## 不同JDK版本的默认垃圾回收器

|JDK版本|默认垃圾回收器|
|---|---|
|JDK 8及以前|Parallel GC|
|JDK 9-10|G1 GC|
|JDK 11+|G1 GC|

## 选择垃圾回收器的考虑因素

### 1. **应用特性**

- **吞吐量优先**：选择Parallel GC
- **低延迟优先**：选择G1 GC或ZGC
- **内存大小**：大内存应用建议G1或ZGC

### 2. **硬件环境**

- **CPU核心数**：多核选择并行回收器
- **内存大小**：大内存选择G1或ZGC
- **响应时间要求**：严格要求选择ZGC

### 3. **性能调优参数示例**

```bash
# G1 GC调优示例
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m
-XX:+G1UseAdaptiveIHOP

# ZGC调优示例
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions
-Xmx8g
```

通过以上方法，你可以准确判断Java项目使用的垃圾回收器类型，并根据项目需求进行相应的调优。