我来详细解释一下synchronized锁升级与降级的机制。

## synchronized锁的演进过程

在JDK 1.6之前，synchronized是重量级锁，性能较差。从JDK 1.6开始，JVM对synchronized进行了大量优化，引入了锁升级机制。

## 锁的四种状态

synchronized锁有四种状态，按性能从优到差排列：

1. **无锁状态** - 对象头markword正常存储对象信息
2. **偏向锁** - 适用于只有一个线程访问的场景
3. **轻量级锁** - 适用于少量线程竞争的场景
4. **重量级锁** - 适用于多线程激烈竞争的场景

## 锁升级过程

### 1. 偏向锁阶段

```java
public class BiasedLockExample {
    private int count = 0;
    
    public synchronized void increment() {
        count++; // 第一次访问，获得偏向锁
    }
}
```

- 当第一个线程访问时，JVM在对象头中记录线程ID
- 后续该线程再次访问时，直接检查线程ID，无需CAS操作
- 几乎无性能开销

### 2. 轻量级锁阶段

```java
// 当第二个线程尝试访问时
Thread t1 = new Thread(() -> obj.increment());
Thread t2 = new Thread(() -> obj.increment());
t1.start();
t2.start(); // 偏向锁升级为轻量级锁
```

- 有竞争时，偏向锁撤销，升级为轻量级锁
- 线程通过CAS操作获取锁
- 自旋等待一定次数

### 3. 重量级锁阶段

```java
// 当自旋次数超过阈值或等待线程过多时
for(int i = 0; i < 100; i++) {
    new Thread(() -> obj.increment()).start();
}
// 轻量级锁升级为重量级锁
```

## 对象头结构变化

```
无锁状态：
|------|------|------|------|------|------|------|------|
| 哈希码  | GC分代年龄 | 0 | 01 |

偏向锁状态：
|------|------|------|------|------|------|------|------|
| 线程ID | epoch | GC分代年龄 | 1 | 01 |

轻量级锁：
|------|------|------|------|------|------|------|------|
| 栈帧中锁记录指针           | 00 |

重量级锁：
|------|------|------|------|------|------|------|------|
| 重量级锁指针              | 10 |
```

## 锁降级机制

**重要概念**：synchronized锁是**不可逆的**，即锁只能升级，不能降级。

```java
public class LockUpgradeExample {
    private final Object lock = new Object();
    
    public void testLockUpgrade() {
        // 1. 第一次访问 - 偏向锁
        synchronized(lock) {
            System.out.println("偏向锁");
        }
        
        // 2. 多线程竞争 - 升级为轻量级锁
        Thread t1 = new Thread(() -> {
            synchronized(lock) {
                System.out.println("轻量级锁");
            }
        });
        t1.start();
        
        // 3. 激烈竞争 - 升级为重量级锁
        // 一旦升级为重量级锁，即使竞争减少也不会降级
    }
}
```

## 相关优化技术

### 1. 锁消除

```java
public String concatString(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1); // JVM检测到sb不会逃逸，消除synchronized
    sb.append(s2);
    return sb.toString();
}
```

### 2. 锁粗化

```java
// 优化前
public void method() {
    synchronized(lock) { operation1(); }
    synchronized(lock) { operation2(); }
    synchronized(lock) { operation3(); }
}

// JVM优化后
public void method() {
    synchronized(lock) {
        operation1();
        operation2(); 
        operation3();
    }
}
```

### 3. 自适应自旋

```java
// JVM会根据历史自旋成功率调整自旋次数
while(!tryAcquireLock()) {
    // 自旋等待，次数动态调整
    spin();
}
```

## 性能对比

|锁类型|获取锁耗时|适用场景|
|---|---|---|
|偏向锁|~2-3ns|单线程访问|
|轻量级锁|~20-30ns|少量线程短时竞争|
|重量级锁|~200-300ns|多线程激烈竞争|

## 实际应用建议

1. **避免在循环中使用synchronized**
2. **减小同步代码块范围**
3. **考虑使用并发集合类替代synchronized**
4. **在高并发场景下考虑使用Lock接口**

通过这种渐进式的锁升级机制，synchronized在保证线程安全的同时，大大提升了性能，这也是为什么在现代Java应用中，synchronized仍然是首选的同步机制之一。