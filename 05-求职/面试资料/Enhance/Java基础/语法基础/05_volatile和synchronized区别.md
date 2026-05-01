# volatile和synchronized区别

## volatile和synchronized的区别

### 1. 基本概念差异

**volatile**是一个关键字修饰符，用于修饰变量，确保变量的==可见性和有序性==。

**synchronized**是一个关键字，用于修饰方法或代码块，提供==互斥锁==机制，确保线程安全。

### 2. 功能对比

|特性|volatile|synchronized|
|---|---|---|
|**原子性**|❌ 不保证|✅ 保证|
|**可见性**|✅ 保证|✅ 保证|
|**有序性**|✅ 禁止指令重排|✅ 保证|
|**互斥性**|❌ 不提供|✅ 提供|

### 3. 使用场景

**volatile适用于：**

```java
// 1. 状态标记
public class ThreadSafeCounter {
    private volatile boolean running = true;
    
    public void stop() {
        running = false; // 立即对所有线程可见
    }
    
    public void doWork() {
        while (running) {
            // 执行工作
        }
    }
}

// 2. 单例模式双重检查锁定
public class Singleton {
    private static volatile Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 防止指令重排
                }
            }
        }
        return instance;
    }
}
```

**synchronized适用于：**

```java
// 1. 修饰方法
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++; // 原子性操作
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// 2. 修饰代码块
public class BankAccount {
    private double balance;
    private final Object lock = new Object();
    
    public void transfer(double amount) {
        synchronized (lock) {
            // 复合操作需要原子性
            balance -= amount;
            // 其他相关操作
        }
    }
}
```

### 4. 性能差异

**volatile**：

- 轻量级，不会引起线程阻塞
- 仅影响内存可见性，性能开销小
- 适合高频读写的简单变量

**synchronized**：

- 重量级，可能导致线程阻塞和上下文切换
- 涉及锁的获取和释放，性能开销较大
- 适合复杂的临界区保护

### 5. 底层实现原理

**volatile实现：**

- 通过==内存屏障==（Memory Barrier）确保可见性
- 禁止编译器和CPU的指令重排序
- 每次读写都直接操作主内存

**synchronized实现：**

- ==基于JVM的监视器锁（Monitor）==
- 使用对象头中的Mark Word存储锁信息
- 涉及偏向锁、轻量级锁、重量级锁的升级过程

### 6. 常见误区

==volatile 不能保证原子性！==

```java
// 错误示例：volatile不能保证原子性
public class VolatileCounter {
    private volatile int count = 0;
    
    public void increment() {
        count++; // 非原子操作！包含读取、加1、写回三步
    }
}

// 正确做法：使用synchronized或原子类
public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet(); // 原子操作
    }
}
```

### 7. 选择建议

**使用volatile当：**

- 变量的写入不依赖当前值
- 变量不需要与其他状态变量共同参与不变性条件
- 访问变量时不需要加锁

**使用synchronized当：**

- 需要原子性操作
- 多个变量需要同时更新
- 复杂的业务逻辑需要线程安全保护

### 8. 相关知识扩展

现代Java还提供了其他并发工具：

- **Lock接口**：提供比synchronized更灵活的锁机制
- **原子类**：如AtomicInteger，提供无锁的原子操作
- **并发集合**：如ConcurrentHashMap，高性能的线程安全集合

理解这些概念的区别和适用场景，是编写高质量并发代码的关键。