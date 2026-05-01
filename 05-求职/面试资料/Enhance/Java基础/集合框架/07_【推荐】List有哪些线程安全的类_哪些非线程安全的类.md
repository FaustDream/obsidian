# List有哪些线程安全的类，哪些非线程安全的类

## Java中List的线程安全性分类

### 线程安全的List实现类

**1. Vector**

- 最早的线程安全List实现
- 所有方法都使用synchronized关键字同步
- 性能较差，现在很少使用

```java
Vector<String> vector = new Vector<>();
vector.add("item"); // 线程安全
```

**2. Collections.synchronizedList()**

- 通过包装器模式实现线程安全
- 将非线程安全的List包装成线程安全的

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

**3. CopyOnWriteArrayList**

- 写时复制机制，读操作无锁
- 适用于读多写少的场景
- 写操作会复制整个数组，内存开销大

```java
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
```

### 非线程安全的List实现类

**1. ArrayList**

- 最常用的List实现
- 基于动态数组，查找快，插入删除慢
- 多线程环境下可能出现数据不一致或无限循环

**2. LinkedList**

- 基于双向链表实现
- 插入删除快，查找慢
- 同样不是线程安全的

**3. AttributeList**

- 继承自ArrayList
- 主要用于JMX相关操作

## 扩展知识点

### 为什么ArrayList不是线程安全的？

1. **扩容机制问题**：多线程同时扩容可能导致数据丢失
2. **索引计算问题**：size字段的非原子性操作
3. **数组赋值问题**：elementData数组的并发修改

### 如何选择合适的线程安全List？

```java
// 读多写少场景
CopyOnWriteArrayList<String> readHeavyList = new CopyOnWriteArrayList<>();

// 读写平衡场景
List<String> balancedList = Collections.synchronizedList(new ArrayList<>());

// 注意：遍历时仍需手动同步
synchronized(balancedList) {
    for(String item : balancedList) {
        // 处理逻辑
    }
}
```

### 性能对比

|实现类|读操作|写操作|内存占用|适用场景|
|---|---|---|---|---|
|ArrayList|快|快|低|单线程|
|Vector|慢|慢|低|遗留代码|
|SynchronizedList|中等|中等|低|读写平衡|
|CopyOnWriteArrayList|很快|很慢|高|读多写少|

### 实际开发建议

1. **优先使用ArrayList**配合外部同步机制
2. **避免使用Vector**，性能差且功能可替代
3. **读多写少场景**使用CopyOnWriteArrayList
4. **需要线程安全且读写平衡**时使用Collections.synchronizedList()

这样的分类和选择能帮助你在不同场景下做出最优的技术决策。