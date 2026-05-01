当然知道LinkedHashMap！让我来详细介绍一下这个非常实用的数据结构。

## LinkedHashMap是什么？

LinkedHashMap是HashMap的一个子类，它在HashMap的基础上维护了一个**双向链表**，用来记录元素的插入顺序或访问顺序。

## 核心特性

### 1. 保持顺序

- **插入顺序**：默认情况下，LinkedHashMap会保持元素的插入顺序
- **访问顺序**：==通过构造参数可以设置为访问顺序（LRU顺序）==

```java
// 插入顺序（默认）
Map<String, Integer> insertionOrder = new LinkedHashMap<>();

// 访问顺序
Map<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
```

### 2. 性能特征

- 查找、插入、删除：**O(1)** 时间复杂度
- 遍历：**O(n)** 时间复杂度，且按照链表顺序遍历，不需要扫描空bucket

## 主要应用场景

### 1. LRU缓存实现

这是LinkedHashMap最经典的应用：

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(16, 0.75f, true); // 访问顺序
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity; // 超过容量时移除最老的元素
    }
}
```

### 2. 保持插入顺序的Map

当你需要一个既有HashMap性能又能保持顺序的Map时：

```java
Map<String, String> orderedMap = new LinkedHashMap<>();
orderedMap.put("first", "1");
orderedMap.put("second", "2");
orderedMap.put("third", "3");

// 遍历时会按照插入顺序：first -> second -> third
```

### 3. 配置文件处理

处理需要保持原始顺序的配置项。

## 内部实现原理

LinkedHashMap在HashMap的基础上：

- 每个Entry节点增加了`before`和`after`指针
- 维护了一个双向链表的头节点`header`
- 重写了`transfer()`等方法来维护链表结构

## 与其他Map的对比

|特性|HashMap|LinkedHashMap|TreeMap|
|---|---|---|---|
|顺序保证|无|插入/访问顺序|自然/比较器顺序|
|时间复杂度|O(1)|O(1)|O(log n)|
|内存开销|低|中等|高|
|遍历性能|O(capacity)|O(size)|O(size)|

## 相关扩展知识

### 1. 为什么LinkedHashMap遍历更快？

- HashMap遍历需要扫描整个bucket数组，包括空位置
- LinkedHashMap直接沿着链表遍历，只访问实际存在的元素

### 2. 线程安全问题

LinkedHashMap同样不是线程安全的，多线程环境需要：

```java
Map<K, V> synchronizedMap = Collections.synchronizedMap(new LinkedHashMap<>());
// 或使用ConcurrentHashMap（但会失去顺序保证）
```

### 3. 内存考虑

LinkedHashMap比HashMap多消耗约20%的内存，因为需要维护额外的指针。

LinkedHashMap是一个非常实用的数据结构，特别是在需要兼顾性能和顺序的场景下，它提供了优雅的解决方案。