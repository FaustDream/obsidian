ThreadLocal 在 Java 中使用了一种特殊的哈希冲突解决方案，主要基于**线性探测法（Linear Probing）**，但有一些独特的设计：

## 核心机制

### 1. **ThreadLocalMap 的存储结构**

ThreadLocal 内部使用 `ThreadLocalMap` 来存储数据，这是一个定制的哈希表：

- 使用 `Entry[] table` 数组存储键值对
- Entry 继承自 `WeakReference<ThreadLocal<?>>`，key 是 ThreadLocal 对象的弱引用

### 2. **特殊的哈希函数**

```java
private static final int HASH_INCREMENT = 0x61c88647;
private static AtomicInteger nextHashCode = new AtomicInteger();

private final int threadLocalHashCode = nextHashCode.getAndAdd(HASH_INCREMENT);
```

这个魔数 `0x61c88647` 是**黄金分割比例**的倍数，能够在2的幂次大小的哈希表中产生非常均匀的分布，大大减少冲突概率。

### 3. **线性探测冲突解决**

当发生哈希冲突时：

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1); // 计算初始位置

    // 线性探测
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        
        if (k == key) {
            e.value = value;  // 找到相同key，更新值
            return;
        }
        
        if (k == null) {
            replaceStaleEntry(key, value, i);  // 清理过期entry
            return;
        }
    }
    
    tab[i] = new Entry(key, value);  // 在空位置插入新entry
}

private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);  // 循环到数组开头
}
```

## 关键特性

### 4. **自动清理机制**

- 由于使用弱引用，当 ThreadLocal 对象被GC回收时，key 会变成 null
- 在 get/set 操作过程中会自动清理这些"过期"的 Entry
- 这避免了内存泄漏问题

### 5. **负载因子控制**

- 默认负载因子为 2/3（相比 HashMap 的 0.75 更保守）
- 当元素数量超过阈值时会进行 rehash，扩容为原来的 2 倍

## 优势

1. **冲突少**：黄金分割哈希函数分布均匀
2. **缓存友好**：线性探测具有良好的空间局部性
3. **内存安全**：弱引用 + 自动清理防止内存泄漏
4. **简单高效**：相比链表法，线性探测实现更简单，性能更好

这种设计特别适合 ThreadLocal 的使用场景，因为每个线程的 ThreadLocal 变量通常数量不多，冲突概率本身就比较低。