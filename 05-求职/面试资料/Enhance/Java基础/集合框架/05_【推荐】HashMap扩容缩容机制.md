# HashMap扩容缩容机制

HashMap的扩容和缩容机制是Java集合框架中的核心知识点，我来详细解释一下。

## 扩容机制

### 触发条件

HashMap扩容的核心触发条件是**负载因子（Load Factor）**：

- 默认负载因子：0.75
- 触发条件：`size >= threshold`，其中`threshold = capacity × loadFactor`
- 例如：初始容量16，负载因子0.75，当元素个数达到12时触发扩容

### 扩容过程

1. **容量翻倍**：新容量 = 旧容量 × 2
2. **创建新数组**：创建新的Node数组
3. **重新哈希（rehash）**：遍历旧数组，重新计算每个元素的位置
4. **优化策略**：JDK 1.8引入了优化，元素要么保持原位置，要么移动到"原位置+旧容量"的位置

```java
// JDK 1.8 扩容核心逻辑示例
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int newCap = oldCap << 1; // 容量翻倍
    
    // 创建新数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    
    // 重新分配元素...
}
```

### 位置计算优化

JDK 1.8的巧妙设计：

- 通过`(e.hash & oldCap) == 0`判断元素新位置
- 如果为0，位置不变
- 如果为1，新位置 = 原位置 + 旧容量

## 缩容机制

**重要提醒**：HashMap**没有自动缩容机制**！

### 为什么不缩容？

1. **性能考虑**：缩容需要重新哈希，开销大
2. **使用场景**：HashMap通常用于数据量相对稳定的场景
3. **内存管理**：Java依赖GC回收不再使用的对象

### 手动缩容方案

如果确实需要缩容，可以：

```java
// 创建新的HashMap并复制数据
Map<String, String> newMap = new HashMap<>(oldMap);
oldMap = newMap;
```

## 性能影响分析

### 扩容成本

- **时间复杂度**：O(n)，需要遍历所有元素
- **空间复杂度**：临时需要2倍内存空间
- **频繁扩容**：如果初始容量过小，会导致多次扩容

### 最佳实践

1. **预估容量**：根据数据量设置合理的初始容量

```java
// 如果预计存储1000个元素
int initialCapacity = (int)(1000 / 0.75) + 1;
Map<String, String> map = new HashMap<>(initialCapacity);
```

2. **选择合适的负载因子**：
    - ==0.75是时间和空间的平衡点==
    - 更小的负载因子：更少冲突，更多内存
    - 更大的负载因子：更多冲突，更少内存

## 相关扩展知识

### 与其他Map实现的对比

- **LinkedHashMap**：继承HashMap，扩容机制相同
- **ConcurrentHashMap**：分段扩容，支持并发
- **TreeMap**：基于红黑树，没有扩容概念

### 哈希冲突处理

HashMap使用链表+红黑树处理冲突：

- 链表长度≤8：使用链表
- 链表长度>8且数组长度≥64：转换为红黑树
- 红黑树节点≤6：退化为链表

### 线程安全问题

HashMap扩容在多线程环境下可能导致：

- 数据丢失
- 死循环（JDK 1.7）
- 数据不一致

解决方案：使用ConcurrentHashMap或Collections.synchronizedMap()

这个机制的设计体现了Java在性能和内存使用之间的权衡，是面试中经常考查的重点知识。