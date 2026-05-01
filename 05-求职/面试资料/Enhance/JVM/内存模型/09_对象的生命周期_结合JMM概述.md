# 对象的生命周期，结合JMM概述

## 对象的生命周期与JMM概述

### 对象生命周期的完整流程

**1. 对象创建阶段**

- **类加载检查**：JVM首先检查new指令的参数是否能在常量池中定位到类的符号引用，并检查类是否已被加载、解析和初始化
- **内存分配**：在Java堆中为新对象分配内存空间，分配方式取决于垃圾收集器类型（指针碰撞或空闲列表）
- **内存初始化**：将分配的内存空间初始化为零值，保证对象的实例字段在不赋值时有确定的初始值
- **对象头设置**：设置对象的Mark Word（包含哈希码、GC分代年龄、锁状态等）和类型指针

**2. 对象使用阶段**

- 对象通过引用被程序访问和操作
- 在此阶段，对象可能被多个线程同时访问，涉及JMM的可见性和有序性问题

**3. 对象销毁阶段**

- **可达性分析**：当对象不再被GC Roots引用时，被标记为可回收
- **finalize()执行**：如果对象重写了finalize()方法，会被放入F-Queue队列等待执行
- **内存回收**：垃圾收集器回收对象占用的内存空间

### JMM与对象生命周期的关系

**1. 对象创建时的内存可见性**

```java
public class ObjectCreation {
    private static MyObject instance;
    
    public static MyObject getInstance() {
        if (instance == null) {
            synchronized(ObjectCreation.class) {
                if (instance == null) {
                    instance = new MyObject(); // 可能发生重排序
                }
            }
        }
        return instance;
    }
}
```

在对象创建过程中，JMM需要保证：

- 对象的构造过程不会被重排序到构造函数之外
- 对象引用的赋值操作对其他线程的可见性

**2. 对象字段的可见性保证**

```java
public class VisibilityExample {
    private volatile boolean initialized = false;
    private String data;
    
    public void initialize() {
        data = "initialized"; // 1
        initialized = true;   // 2 - volatile写
    }
    
    public String getData() {
        if (initialized) {    // 3 - volatile读
            return data;      // 4
        }
        return null;
    }
}
```

通过volatile关键字，JMM保证了happens-before关系：2 happens-before 3，从而保证4能看到1的结果。

**3. 对象销毁时的内存同步**

```java
public class ResourceManager {
    private volatile boolean shutdown = false;
    private final Object lock = new Object();
    
    public void shutdown() {
        synchronized(lock) {
            // 清理资源
            cleanup();
            shutdown = true; // 确保其他线程能看到shutdown状态
        }
    }
}
```

### 深入理解：JMM的核心概念

**1. 工作内存与主内存**

- 每个线程都有自己的工作内存，存储主内存中变量的副本
- 对象的字段修改首先在工作内存中进行，然后同步到主内存
- 这种设计导致了可见性问题

**2. 内存屏障与对象操作**

- **LoadLoad屏障**：确保load1的数据装载先于load2及后续装载指令
- **StoreStore屏障**：确保store1立刻对其他处理器可见
- **LoadStore屏障**：确保load1的数据装载先于store2及后续存储指令
- **StoreLoad屏障**：确保store1立刻对其他处理器可见，且先于load2

**3. happens-before原则在对象生命周期中的应用**

- 程序顺序规则：线程中的每个操作happens-before该线程中任意后续操作
- 监视器锁规则：unlock操作happens-before后续的lock操作
- volatile变量规则：volatile写happens-before后续的volatile读
- 传递性：如果A happens-before B，B happens-before C，则A happens-before C

### 实际应用场景

**1. 单例模式中的双重检查锁定**

```java
public class Singleton {
    private static volatile Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

volatile确保了instance引用的可见性，防止了指令重排序导致的半初始化对象问题。

**2. 对象池中的线程安全**

```java
public class ObjectPool<T> {
    private final Queue<T> pool = new ConcurrentLinkedQueue<>();
    private volatile int size = 0;
    
    public T acquire() {
        T obj = pool.poll();
        if (obj != null) {
            size--; // 原子性问题需要考虑
        }
        return obj;
    }
    
    public void release(T obj) {
        if (obj != null) {
            pool.offer(obj);
            size++; // 原子性问题需要考虑
        }
    }
}
```

### 性能优化考虑

**1. 对象创建的性能影响**

- 避免在循环中创建大量临时对象
- 使用对象池复用重对象
- 考虑使用StringBuilder代替String连接

**2. JMM相关的性能优化**

- 合理使用volatile，避免过度同步
- 利用final关键字优化字段访问
- 使用无锁数据结构减少同步开销

这样的设计确保了在多线程环境下对象生命周期的正确性和性能，是Java内存模型的核心体现。