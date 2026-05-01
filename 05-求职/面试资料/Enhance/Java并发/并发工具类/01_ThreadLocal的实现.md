# ThreadLocal的实现

ThreadLocal的实现原理主要涉及以下几个核心方面：

## 核心数据结构

==ThreadLocal的实现基于Thread类内部的`ThreadLocalMap`：==

```java
public class Thread {
    // 每个Thread实例都有一个ThreadLocalMap
    ThreadLocal.ThreadLocalMap threadLocals = null;
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```

## ThreadLocalMap的实现

ThreadLocalMap是ThreadLocal的静态内部类，它的实现类似于HashMap但有重要区别：

### 1. Entry结构

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);  // ThreadLocal作为弱引用的key
        value = v;
    }
}
```

### 2. 存储结构

- ==使用数组存储Entry，而不是链表==
- ==采用开放地址法解决哈希冲突==
- ==数组初始容量为16，负载因子为2/3==

## 核心方法实现

### set()方法

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### get()方法

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```

## 内存管理机制

### 弱引用设计

- ThreadLocal作为Entry的key使用弱引用
- 当ThreadLocal对象被回收时，key会变成null
- 这样设计是为了防止ThreadLocal对象无法被回收

### 内存泄漏处理

ThreadLocalMap会在以下时机清理过期Entry：

1. set()操作时遇到过期Entry
2. get()操作时发现过期Entry
3. remove()操作时主动清理
4. rehash()时全面清理

## 哈希算法优化

ThreadLocal使用特殊的哈希算法：

```java
private final int threadLocalHashCode = nextHashCode();
private static final int HASH_INCREMENT = 0x61c88647;

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

这个魔数0x61c88647能够在2的幂次方长度的数组中产生均匀分布，减少哈希冲突。

## 扩容机制

当Entry数量超过threshold（len * 2 / 3）时：

1. 首先尝试清理过期Entry
2. 如果清理后仍然过多，则进行扩容
3. 扩容后重新哈希所有有效Entry

## 相关知识扩展

### 1. InheritableThreadLocal

- 继承自ThreadLocal
- 子线程可以访问父线程的ThreadLocal变量
- 在Thread创建时会复制父线程的inheritableThreadLocals

### 2. 使用场景

- 数据库连接管理
- 用户会话信息存储
- 事务管理
- 日志MDC（Mapped Diagnostic Context）

### 3. 最佳实践

- 使用完毕后及时调用remove()
- 可以声明为static final
- 考虑使用try-finally确保清理

### 4. 性能特点

- 没有锁竞争，线程安全
- 哈希查找，时间复杂度接近O(1)
- 空间复杂度与ThreadLocal变量数量成正比

这种设计既保证了线程安全，又避免了synchronized的性能开销，是Java并发编程中的重要工具。