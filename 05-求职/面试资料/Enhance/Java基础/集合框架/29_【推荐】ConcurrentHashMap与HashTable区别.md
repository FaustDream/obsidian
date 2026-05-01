ConcurrentHashMap与HashTable都是线程安全的Map实现，但它们在设计理念和性能表现上有显著差异：

## 线程安全机制的区别

**HashTable**：

- 使用`synchronized`关键字对整个方法进行同步
- 所有操作都是互斥的，同一时刻只能有一个线程访问
- 粗粒度锁机制，锁定整个数据结构

**ConcurrentHashMap**：

- JDK 1.7：采用分段锁（Segment）技术，将数据分成多个段，每段独立加锁
- JDK 1.8+：使用CAS（Compare-And-Swap）+ synchronized的组合，锁粒度更细

## 性能差异

**HashTable性能特点**：

```java
// HashTable的put方法
public synchronized V put(K key, V value) {
    // 整个方法被锁定，并发性能差
}
```

**ConcurrentHashMap性能特点**：

```java
// JDK 1.8中的put操作，只对需要的桶加锁
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 使用CAS操作，只在必要时加锁
    if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
        if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
            break; // 无锁成功插入
    }
}
```

## 具体技术实现

**分段锁机制（JDK 1.7的ConcurrentHashMap）**：

- 默认16个Segment，每个Segment管理一部分桶
- 不同Segment可以并发访问，提高并发度
- 并发度 = Segment数量

**CAS + synchronized（JDK 1.8+的ConcurrentHashMap）**：

- 桶==为空时==使用CAS操作
- 桶==不为空时==对桶的首节点加synchronized锁
- 锁粒度降低到单个桶级别

## 扩展知识点

**1. 为什么HashTable被淘汰？**

- 性能瓶颈：所有操作串行化
- 现代CPU多核优势无法发挥
- 不支持null值（key和value都不能为null）

**2. ConcurrentHashMap的计数实现**

```java
// JDK 1.8使用CounterCell数组 + baseCount
private transient volatile CounterCell[] counterCells;
private transient volatile long baseCount;

// size()方法实现
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 : (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE : (int)n);
}
```

**3. 读操作的优化**

- ConcurrentHashMap的读操作（get）通常不需要加锁
- 使用volatile保证可见性
- HashTable的读操作也需要synchronized，效率低

**4. 迭代器的差异**

- HashTable：fail-fast迭代器，并发修改抛出ConcurrentModificationException
- ConcurrentHashMap：弱一致性迭代器，允许并发修改

**5. 适用场景**

- **HashTable**：已基本不推荐使用，可考虑Collections.synchronizedMap()
- **ConcurrentHashMap**：高并发读写场景的首选，如缓存实现、计数器等

**总结**：ConcurrentHashMap通过更细粒度的锁机制和无锁技术，在保证线程安全的同时大幅提升了并发性能，是现代Java应用中处理并发Map操作的标准选择。