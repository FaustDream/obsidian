## 什么是AQS？

**AQS（AbstractQueuedSynchronizer）** 是Java并发包`java.util.concurrent`的核心基础框架，是一个用于构建锁和同步器的==抽象队列同步器==。

### AQS的核心特点：

**1. 设计思想**

- 基于模板方法模式设计
- 使用一个`volatile int state`变量表示同步状态
- 采用FIFO队列管理等待线程
- 支持独占模式和共享模式两种同步方式

**2. 核心组件**

```java
// 同步状态
private volatile int state;

// 等待队列的头节点
private transient volatile Node head;

// 等待队列的尾节点  
private transient volatile Node tail;
```

**3. 工作原理**

- 线程尝试获取资源，成功则直接返回
- 失败则加入等待队列，通过自旋或阻塞等待
- 资源释放时唤醒队列中的后继节点

## AQS的常用实现

### 1. **ReentrantLock（可重入锁）**

```java
// 独占模式的典型实现
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock();
}
```

### 2. **CountDownLatch（倒计时门闩）**

```java
// 共享模式实现，等待多个线程完成
CountDownLatch latch = new CountDownLatch(3);
// 等待线程
latch.await();
// 工作线程
latch.countDown();
```

### 3. **Semaphore（信号量）**

```java
// 控制同时访问资源的线程数量
Semaphore semaphore = new Semaphore(2);
semaphore.acquire(); // 获取许可
try {
    // 访问资源
} finally {
    semaphore.release(); // 释放许可
}
```

### 4. **CyclicBarrier（循环屏障）**

```java
// 让多个线程互相等待到达某个屏障点
CyclicBarrier barrier = new CyclicBarrier(3);
barrier.await(); // 等待其他线程
```

### 5. **ReadWriteLock（读写锁）**

```java
// ReentrantReadWriteLock基于AQS实现
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();   // 读锁（共享）
rwLock.writeLock().lock();  // 写锁（独占）
```

## 扩展知识点

### AQS的两种模式对比：

|特性|独占模式（Exclusive）|共享模式（Shared）|
|---|---|---|
|资源获取|同时只有一个线程|可以多个线程同时获取|
|典型应用|ReentrantLock|CountDownLatch, Semaphore|
|核心方法|tryAcquire/tryRelease|tryAcquireShared/tryReleaseShared|

### 面试延伸问题：

1. **AQS如何实现公平锁和非公平锁？**
2. **为什么AQS使用双向链表而不是单向链表？**
3. **AQS中的自旋优化策略是什么？**
4. **如何基于AQS实现自定义同步器？**

AQS作为并发编程的基石，理解其原理对掌握Java并发编程至关重要。通过学习这些实现类，可以更好地理解不同同步场景的解决方案。