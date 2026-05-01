# Java集合的整个体系和继承关系

Java集合框架是Java编程中最核心的内容之一，它提供了一套完整的数据结构实现。让我详细介绍整个集合体系的架构和继承关系。

## 集合框架的顶层接口

### 1. Collection接口

`Collection`是所有集合类的根接口，定义了集合的基本操作：

- `add(E e)` - 添加元素
- `remove(Object o)` - 删除元素
- `size()` - 获取集合大小
- `isEmpty()` - 判断是否为空
- `contains(Object o)` - 判断是否包含某元素
- `iterator()` - 获取迭代器

### 2. Map接口

`Map`是键值对集合的根接口，与Collection平级，定义了映射关系的基本操作：

- `put(K key, V value)` - 存储键值对
- `get(Object key)` - 根据键获取值
- `remove(Object key)` - 删除键值对
- `keySet()` - 获取所有键的集合
- `values()` - 获取所有值的集合

## Collection接口的三大子接口

### 1. List接口 - 有序可重复

**特点**：有序、可重复、支持索引访问

**主要实现类**：

- **ArrayList**：基于动态数组实现，查询快O(1)，插入删除慢O(n)
- **LinkedList**：基于双向链表实现，插入删除快O(1)，查询慢O(n)
- **Vector**：线程安全的动态数组，性能较差，现已很少使用
- **Stack**：继承自Vector，实现栈的LIFO特性

### 2. Set接口 - 无序不可重复

**特点**：不允许重复元素

**主要实现类**：

- **HashSet**：基于HashMap实现，==无序==，O(1)时间复杂度
- **LinkedHashSet**：继承自HashSet，==维护插入顺序==
- **TreeSet**：基于红黑树实现，有序，O(log n)时间复杂度

### 3. Queue接口 - 队列

**特点**：FIFO（先进先出）

**主要实现类**：

- **ArrayDeque**：基于数组的双端队列
- **PriorityQueue**：基于堆的优先队列
- **LinkedList**：也实现了Queue接口

## Map接口的主要实现类

### 1. HashMap

- 基于哈希表实现
- 允许null键和null值
- 非线程安全
- 平均时间复杂度O(1)

### 2. LinkedHashMap

- 继承自HashMap
- 维护插入顺序或访问顺序
- 可用于实现LRU缓存

### 3. TreeMap

- 基于红黑树实现
- 有序映射
- 时间复杂度O(log n)

### 4. ConcurrentHashMap

- 线程安全的HashMap
- 使用分段锁技术
- 高并发性能优异

### 5. Hashtable

- 线程安全但性能较差
- 不允许null键和null值
- 现已很少使用

## 继承关系图示

```
                    Iterable
                       |
                  Collection
                 /      |      \
              List    Set     Queue
             /  |      /|\      |  \
      ArrayList |  HashSet|  ArrayDeque \
     LinkedList |     |   |      |    PriorityQueue
        Vector  |     |TreeSet   |
         Stack  |LinkedHashSet   |
                |                |
           (LinkedList implements both List and Queue)

                    Map
                 /   |   \
           HashMap   |   TreeMap
              |      |      |
      LinkedHashMap  |  ConcurrentHashMap
                     |
                Hashtable
```

## 设计模式在集合中的应用

### 1. 迭代器模式

所有集合都实现了`Iterable`接口，提供统一的遍历方式：

```java
for (String item : collection) {
    // 处理元素
}
```

### 2. 装饰器模式

`Collections`工具类提供了多种装饰器：

- `Collections.synchronizedList()` - 线程安全装饰
- `Collections.unmodifiableList()` - 不可修改装饰

### 3. 适配器模式

`Arrays.asList()`将数组适配为List接口

## 性能特点总结

|集合类型|查询|插入|删除|有序性|线程安全|
|---|---|---|---|---|---|
|ArrayList|O(1)|O(n)|O(n)|有序|否|
|LinkedList|O(n)|O(1)|O(1)|有序|否|
|HashSet|O(1)|O(1)|O(1)|无序|否|
|TreeSet|O(log n)|O(log n)|O(log n)|有序|否|
|HashMap|O(1)|O(1)|O(1)|无序|否|
|TreeMap|O(log n)|O(log n)|O(log n)|有序|否|

## 选择建议

1. **需要频繁随机访问**：选择ArrayList
2. **需要频繁插入删除**：选择LinkedList
3. **需要去重且无序**：选择HashSet
4. **需要去重且有序**：选择TreeSet
5. **需要键值对存储**：选择HashMap
6. **需要线程安全**：选择ConcurrentHashMap
7. **需要排序的键值对**：选择TreeMap

这个集合框架的设计体现了面向对象设计的精髓，通过接口定义规范，通过不同实现类提供不同的性能特性，开发者可以根据具体需求选择最合适的集合类型。