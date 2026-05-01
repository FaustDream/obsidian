# fullgc和younggc区别

## Full GC 和 Young GC 的区别

### 基本概念

**Young GC（Minor GC）**：

- 只针对年轻代（Young Generation）进行的垃圾回收
- 回收范围包括Eden区和Survivor区
- 发生频率较高，但耗时较短

**Full GC（Major GC）**：

- 对整个堆内存进行的垃圾回收
- 回收范围包括年轻代、老年代和永久代/元空间
- 发生频率较低，但耗时较长

### 详细对比

|维度|Young GC|Full GC|
|---|---|---|
|**触发条件**|Eden区空间不足|老年代空间不足、永久代/元空间不足、System.gc()调用|
|**回收范围**|年轻代（Eden + Survivor）|整个堆内存 + 永久代/元空间|
|**执行频率**|频繁（秒级或更频繁）|较少（分钟级或更长）|
|**停顿时间**|短（几毫秒到几十毫秒）|长（几百毫秒到几秒）|
|**性能影响**|较小|较大|

### 触发机制详解

**Young GC触发时机**：

```java
// 当Eden区满了，就会触发Young GC
Object[] largeArray = new Object[1000000]; // 可能触发Young GC
```

**Full GC触发时机**：

1. 老年代空间不足
2. 永久代/元空间不足
3. 调用System.gc()
4. CMS GC出现promotion failed
5. 统计得到的Young GC晋升到老年代的平均大小大于老年代的剩余空间

### 对象晋升流程

```
新对象 → Eden区 → Young GC → 
存活对象 → Survivor0/1 → 多次GC后 → 老年代
```

### 性能优化建议

**减少Full GC的策略**：

1. **合理设置堆大小**：
    
    ```bash
    -Xms2g -Xmx2g  # 设置初始和最大堆大小
    -XX:NewRatio=2  # 设置年轻代与老年代比例
    ```
    
2. **避免大对象直接进入老年代**：
    
    ```java
    // 避免创建大数组或大字符串
    List<String> list = new ArrayList<>(); // 好的做法
    String[] bigArray = new String[1000000]; // 可能直接进入老年代
    ```
    
3. **选择合适的垃圾回收器**：
    
    - G1GC：低延迟应用
    - CMS：响应时间敏感
    - Parallel GC：吞吐量优先

### 监控和调优

**JVM参数监控**：

```bash
-XX:+PrintGC                    # 打印GC信息
-XX:+PrintGCDetails            # 打印GC详细信息
-XX:+PrintGCTimeStamps         # 打印GC时间戳
-XX:+PrintGCApplicationStoppedTime  # 打印应用停顿时间
```

**关键指标**：

- Young GC频率和耗时
- Full GC频率和耗时
- 堆内存使用率
- 晋升率（年轻代到老年代的对象转移速率）

### 实际案例分析

**正常情况**：

```
Young GC: 10ms, 每30秒一次
Full GC: 200ms, 每30分钟一次
```

**异常情况**：

```
Young GC: 50ms, 每5秒一次  → 堆太小或对象创建过快
Full GC: 2000ms, 每分钟一次 → 内存泄漏或老年代过小
```

### 扩展知识

**不同垃圾回收器的GC行为**：

- **Serial GC**：单线程，适合小应用
- **Parallel GC**：多线程，适合吞吐量优先
- **CMS GC**：并发标记清除，适合低延迟
- **G1 GC**：分代分区，平衡吞吐量和延迟
- **ZGC/Shenandoah**：超低延迟垃圾回收器

理解这些区别对于JVM调优和解决性能问题至关重要，特别是在高并发和大内存的应用场景中。