# Java有哪些控制资源竞争的方法

Java中控制资源竞争的方法主要有以下几种：

## 1. 同步机制（Synchronization）

### synchronized关键字

```java
// 同步方法
public synchronized void method() {
    // 临界区代码
}

// 同步代码块
public void method() {
    synchronized(this) {
        // 临界区代码
    }
}
```

### ReentrantLock（可重入锁）

```java
private final ReentrantLock lock = new ReentrantLock();

public void method() {
    lock.lock();
    try {
        // 临界区代码
    } finally {
        lock.unlock();
    }
}
```

## 2. 原子操作类（Atomic Classes）

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // 原子性递增
counter.compareAndSet(expectedValue, newValue); // CAS操作
```

## 3. 并发集合（Concurrent Collections）

- `ConcurrentHashMap`：线程安全的HashMap
- `CopyOnWriteArrayList`：读多写少场景
- `BlockingQueue`：阻塞队列，用于生产者-消费者模式

## 4. 线程池（Thread Pools）

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> {
    // 任务代码
});
```

## 5. 信号量（Semaphore）

```java
Semaphore semaphore = new Semaphore(3); // 允许3个线程同时访问
semaphore.acquire(); // 获取许可
try {
    // 受限资源访问
} finally {
    semaphore.release(); // 释放许可
}
```

## 6. 条件变量（Condition）

```java
private final Lock lock = new ReentrantLock();
private final Condition condition = lock.newCondition();

public void waitForCondition() throws InterruptedException {
    lock.lock();
    try {
        while (!conditionMet) {
            condition.await();
        }
    } finally {
        lock.unlock();
    }
}
```

## 7. 读写锁（ReadWriteLock）

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// 读操作
readLock.lock();
try {
    // 读取数据
} finally {
    readLock.unlock();
}
```

## 8. ThreadLocal

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("thread-specific-value");
String value = threadLocal.get();
```

## 9. volatile关键字

```java
private volatile boolean flag = false;
// 保证可见性，防止指令重排序
```

## 10. 无锁编程（Lock-free Programming）

使用CAS（Compare-And-Swap）操作实现无锁数据结构。

## 性能考虑

1. **synchronized** vs **ReentrantLock**：ReentrantLock提供更多功能，但synchronized在JVM层面优化更好
2. **粗粒度锁** vs **细粒度锁**：需要在并发性能和复杂度之间平衡
3. **锁竞争**：应该尽量减少锁的持有时间

## 最佳实践

1. 优先使用高级并发工具（如并发集合）
2. 避免过度同步
3. 使用不可变对象
4. 正确处理异常，确保锁的释放
5. 避免死锁（按相同顺序获取锁）

这些方法可以根据具体场景选择使用，关键是要理解每种方法的适用场景和性能特点。