# CPU占有率飙升，怎么排查

CPU占用率飙升时，我会按照以下系统化的排查思路来定位问题：

## 1. 快速定位问题进程

**使用系统工具快速识别**

```bash
# 查看CPU使用率最高的进程
top -c
# 或者使用htop（更直观）
htop
# 查看Java进程的CPU使用情况
jps -v | grep your-app
```

**重点关注指标**

- 哪个进程CPU使用率最高
- 是否是我们的Java应用进程
- CPU使用率是持续高还是间歇性飙升

## 2. Java应用层面排查

**查看线程CPU使用情况**

```bash
# 获取进程ID
jps
# 查看线程级别的CPU使用率
top -H -p <pid>
# 将高CPU使用的线程ID转换为16进制
printf "%x\n" <thread_id>
```

**生成并分析线程堆栈**

```bash
# 生成线程转储
jstack <pid> > thread_dump.txt
# 查找对应线程的堆栈信息
grep -A 20 "nid=0x<hex_thread_id>" thread_dump.txt
```

## 3. 常见CPU飙升原因分析

**死循环或无限循环**

```java
// 典型问题代码
while(true) {
    // 没有适当的sleep或wait
    processData();
}

// 正确做法
while(running) {
    processData();
    Thread.sleep(100); // 或使用wait/notify
}
```

**频繁的GC活动**

```bash
# 监控GC情况
jstat -gc <pid> 250 # 每250ms输出一次GC信息
# 查看GC日志
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps
```

**正则表达式性能问题**

```java
// 问题代码：每次都编译正则
String.matches("complex.*regex", input);

// 优化：预编译正则
private static final Pattern PATTERN = Pattern.compile("complex.*regex");
PATTERN.matcher(input).matches();
```

## 4. 深入分析工具

**使用JProfiler或JVisualVM**

- 实时监控CPU使用率
- 分析热点方法
- 查看方法调用树

**使用火焰图分析**

```bash
# 生成火焰图（需要安装相关工具）
java -XX:+PreserveFramePointer -jar your-app.jar &
perf record -F 99 -p <pid> -g -- sleep 30
perf script | stackcollapse-perf.pl | flamegraph.pl > cpu.svg
```

## 5. 应用代码层面排查

**检查高频调用的方法**

- 数据库查询是否有N+1问题
- 缓存是否失效导致频繁计算
- 是否有大量的字符串拼接操作

**并发问题排查**

```java
// 检查是否有锁竞争
// 使用jstack查看BLOCKED状态的线程
jstack <pid> | grep -A 5 -B 5 "BLOCKED"
```

## 6. 系统资源层面

**检查系统负载**

```bash
# 查看系统负载
uptime
# 查看CPU详细信息
vmstat 1
# 查看IO等待
iostat -x 1
```

**内存压力检查**

```bash
# 查看内存使用情况
free -h
# 查看swap使用情况
swapon -s
```

## 7. 解决方案和预防措施

**代码层面优化**

- 使用缓存减少重复计算
- 优化算法复杂度
- 合理使用线程池
- 避免在循环中创建大量对象

**JVM参数调优**

```bash
# 调整垃圾回收器
-XX:+UseG1GC
# 设置合适的堆内存大小
-Xms2g -Xmx2g
# 启用JIT编译优化
-XX:+TieredCompilation
```

**监控告警**

- 设置CPU使用率阈值告警
- 配置APM工具进行实时监控
- 建立性能基线对比

## 8. 总结排查流程

1. **快速定位**：使用top/htop找到高CPU进程
2. **线程分析**：使用jstack分析具体线程
3. **代码审查**：检查热点代码逻辑
4. **工具辅助**：使用profiler工具深入分析
5. **系统排查**：检查系统资源和配置
6. **优化实施**：根据分析结果进行针对性优化

这种系统化的排查方法可以帮助我们快速定位CPU飙升的根本原因，并制定有效的解决方案。在实际工作中，建议结合具体的业务场景和系统特点，选择最合适的排查工具和方法。