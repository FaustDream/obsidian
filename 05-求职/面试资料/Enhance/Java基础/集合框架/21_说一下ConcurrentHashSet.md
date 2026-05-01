LinkedHashSet 是 Java 集合框架中的一个重要实现类，我来详细介绍一下：

## 基本概念

LinkedHashSet 继承自 HashSet，同时维护了一个双向链表来记录元素的插入顺序。它结合了 HashSet 的快速查找特性和 LinkedList 的有序特性。

## 核心特点

**唯一性**：不允许重复元素，基于 equals() 和 hashCode() 判断 **有序性**：维护插入顺序，遍历时按照元素添加的先后顺序返回 **性能**：查找、插入、删除操作的时间复杂度为 O(1)

## 底层实现原理

```java
public class LinkedHashSet<E> extends HashSet<E> {
    // 内部使用 LinkedHashMap 实现
    public LinkedHashSet() {
        super(16, .75f, true); // 调用 HashSet 的构造器
    }
}
```

实际上，LinkedHashSet 内部使用 LinkedHashMap 来存储数据，利用了 LinkedHashMap 的 key 来存储元素，value 统一使用一个静态的 PRESENT 对象。

## 与其他集合的对比

**vs HashSet**：

- 相同点：都不允许重复，都基于哈希表
- 不同点：LinkedHashSet 维护插入顺序，HashSet 不保证顺序

**vs TreeSet**：

- LinkedHashSet：插入顺序，O(1) 操作
- TreeSet：自然排序或自定义排序，O(log n) 操作

**vs LinkedList**：

- LinkedHashSet：无重复，快速查找
- LinkedList：允许重复，支持索引访问

## 使用场景

1. **需要去重且保持插入顺序**的场景
2. **缓存实现**中需要 LRU 特性时
3. **数据处理**中需要保持原始顺序的去重操作

## 示例代码

```java
LinkedHashSet<String> set = new LinkedHashSet<>();
set.add("apple");
set.add("banana");
set.add("apple"); // 重复，不会添加
set.add("cherry");

// 输出：apple -> banana -> cherry（保持插入顺序）
for (String item : set) {
    System.out.println(item);
}
```

## 扩展知识点

**线程安全性**：LinkedHashSet 不是线程安全的，多线程环境需要使用 `Collections.synchronizedSet()` 包装或使用 `ConcurrentSkipListSet`

**内存开销**：相比 HashSet 有额外的链表指针开销，每个元素需要额外存储 before 和 after 引用

**迭代器特性**：支持 fail-fast 机制，并发修改时会抛出 `ConcurrentModificationException`

这样的设计使得 LinkedHashSet 在需要去重且保持顺序的场景下成为理想选择。