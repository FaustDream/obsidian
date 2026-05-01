# HashMap和HashTable的区别

HashMap和HashTable的区别可以从以下几个关键维度来理解：

## 1. 线程安全性

- **HashMap**: 线程不安全，多线程环境下需要外部同步
- **HashTable**: 线程安全，所有方法都用synchronized修饰

```java
// HashMap - 需要外部同步
Map<String, String> map = Collections.synchronizedMap(new HashMap<>());

// HashTable - 内置同步
Hashtable<String, String> table = new Hashtable<>();
```

## 2. 空值处理

- **HashMap**: ==允许一个null键和多个null值==
- **HashTable**: ==不允许null键和null值，会抛出NullPointerException==

```java
// HashMap允许null
HashMap<String, String> map = new HashMap<>();
map.put(null, "value");  // 正常
map.put("key", null);    // 正常

// HashTable不允许null
Hashtable<String, String> table = new Hashtable<>();
table.put(null, "value");  // 抛出NullPointerException
```

## 3. 继承关系

- **HashMap**: 继承自AbstractMap类，实现Map接口
- **HashTable**: 继承自Dictionary类，实现Map接口

## 4. 初始容量和扩容

- **HashMap**: 默认初始容量16，扩容时容量翻倍
- **HashTable**: 默认初始容量11，扩容时容量变为原来的2倍+1

## 5. 性能差异

- **HashMap**: 性能更好，因为没有同步开销
- **HashTable**: 性能较差，每个方法都有同步开销

## 6. 引入版本

- **HashMap**: JDK 1.2引入，属于Collections框架
- **HashTable**: JDK 1.0引入，较为古老

## 扩展知识

### 现代替代方案

在实际开发中，如果需要线程安全的Map，==推荐使用：==

- **ConcurrentHashMap**: 分段锁机制，性能远优于HashTable
- **Collections.synchronizedMap()**: 包装现有Map为线程安全版本

### 底层实现差异

- HashMap在JDK 1.8后采用数组+链表+红黑树结构
- HashTable仍然使用传统的数组+链表结构

### 迭代器行为

- **HashMap**: 迭代器是==fail-fast==的，检测到并发修改会抛出==ConcurrentModificationException==
- **HashTable**: 迭代器也是fail-fast的，但由于内置同步，在正确使用下不会出现并发修改

### 实际应用建议

- 单线程环境：使用HashMap
- 多线程环境：使用ConcurrentHashMap而非HashTable
- 需要null值：只能使用HashMap
- 遗留代码兼容：可能需要使用HashTable

这些区别反映了Java集合框架的演进历程，HashMap代表了更现代、更灵活的设计理念。