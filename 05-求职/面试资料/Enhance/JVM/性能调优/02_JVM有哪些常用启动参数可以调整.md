# JVM有哪些常用启动参数可以调整

JVM启动参数是Java应用性能调优的重要手段。我来为你详细介绍常用的JVM启动参数分类：

## 内存相关参数

**堆内存设置：**

- `-Xms`: 初始堆大小，如`-Xms512m`
- `-Xmx`: 最大堆大小，如`-Xmx2g`
- `-Xmn`: 年轻代大小，如`-Xmn256m`
- `-XX:NewRatio=n`: 老年代与年轻代比例，默认为2
- `-XX:SurvivorRatio=n`: Eden区与Survivor区比例，默认为8

**非堆内存设置：**

- `-XX:MetaspaceSize`: 元空间初始大小（JDK8+）
- `-XX:MaxMetaspaceSize`: 元空间最大大小
- `-XX:DirectMemorySize`: 直接内存大小
- `-Xss`: 每个线程栈大小，如`-Xss256k`

## 垃圾回收器参数

**选择垃圾回收器：**

- `-XX:+UseSerialGC`: 串行垃圾回收器
- `-XX:+UseParallelGC`: 并行垃圾回收器
- `-XX:+UseConcMarkSweepGC`: CMS垃圾回收器
- `-XX:+UseG1GC`: G1垃圾回收器
- `-XX:+UseZGC`: ZGC垃圾回收器（JDK11+）

**GC调优参数：**

- `-XX:MaxGCPauseMillis=n`: 最大GC停顿时间目标
- `-XX:ParallelGCThreads=n`: 并行GC线程数
- `-XX:+UseStringDeduplication`: 字符串去重优化

## 性能监控参数

**GC日志：**

- `-XX:+PrintGC`: 打印GC信息
- `-XX:+PrintGCDetails`: 打印详细GC信息
- `-XX:+PrintGCTimeStamps`: 打印GC时间戳
- `-Xloggc:gc.log`: GC日志输出文件

**JVM信息：**

- `-XX:+PrintFlagsFinal`: 打印所有JVM参数
- `-XX:+PrintCommandLineFlags`: 打印命令行参数
- `-XX:+HeapDumpOnOutOfMemoryError`: OOM时生成堆转储

## 编译优化参数

- `-XX:+TieredCompilation`: 启用分层编译
- `-XX:CompileThreshold=n`: 方法编译阈值
- `-XX:+AggressiveOpts`: 启用激进优化
- `-server`: 服务器模式（默认）
- `-client`: 客户端模式

## 实际应用示例

**生产环境典型配置：**

```bash
java -Xms4g -Xmx4g -Xmn2g -Xss256k 
     -XX:+UseG1GC -XX:MaxGCPauseMillis=200 
     -XX:+PrintGCDetails -XX:+PrintGCTimeStamps 
     -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError 
     -jar application.jar
```

**开发环境配置：**

```bash
java -Xms512m -Xmx1g -XX:+UseParallelGC 
     -XX:+PrintGC -jar application.jar
```

## 参数调优建议

1. **内存设置**: 通常设置`-Xms`和`-Xmx`相等，避免堆扩容开销
2. **垃圾回收器选择**: 根据应用特点选择，G1适合大堆内存场景
3. **监控必备**: 生产环境务必开启GC日志和堆转储
4. **分步调优**: 先设置基础参数，再根据监控数据细化调整

这些参数的合理配置能显著提升Java应用的性能表现，需要结合具体业务场景进行调优。