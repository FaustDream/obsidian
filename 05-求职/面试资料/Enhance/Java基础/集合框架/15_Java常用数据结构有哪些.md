# Java常用数据结构有哪些

Java中常用的数据结构可以分为以下几大类：

## 1. 线性数据结构

**数组（Array）**

- 固定大小的连续内存空间
- 访问时间复杂度：O(1)
- 适用于已知大小且需要快速随机访问的场景

**动态数组**

- `ArrayList`：基于数组实现，支持动态扩容
- `Vector`：线程安全的动态数组（已较少使用）
- 查找快O(1)，插入删除慢O(n)

**链表**

- `LinkedList`：双向链表实现
- 插入删除快O(1)，查找慢O(n)
- 适合频繁插入删除的场景

## 2. 栈和队列

**栈（Stack）**

- `Stack`类：后进先出（LIFO）
- 也可用`ArrayDeque`或`LinkedList`实现
- 常用于递归、表达式求值、括号匹配等

**队列（Queue）**

- `Queue`接口的实现：`ArrayDeque`、`LinkedList`
- `PriorityQueue`：优先队列，基于堆实现
- 先进先出（FIFO）或按优先级出队

## 3. 映射结构

**哈希表**

- `HashMap`：基于哈希表，O(1)平均时间复杂度
- `ConcurrentHashMap`：线程安全的哈希表
- `LinkedHashMap`：保持插入顺序的哈希表

**有序映射**

- `TreeMap`：基于红黑树，O(log n)时间复杂度
- 自动按键排序，支持范围查询

## 4. 集合结构

**哈希集合**

- `HashSet`：基于HashMap实现，元素唯一
- `LinkedHashSet`：保持插入顺序的HashSet

**有序集合**

- `TreeSet`：基于TreeMap实现，元素有序且唯一
- 支持快速的范围操作

## 5. 树形结构

虽然Java标准库没有直接的树结构，但TreeMap和TreeSet底层使用红黑树：

- **红黑树**：自平衡二叉搜索树
- 保证O(log n)的查找、插入、删除时间复杂度

## 6. 图结构

Java标准库没有直接的图实现，通常需要：

- 使用邻接矩阵：`int[][]`或`boolean[][]`
- 使用邻接表：`List<List<Integer>>`或`Map<Integer, List<Integer>>`

## 实际应用选择建议

**查找频繁**：HashMap、HashSet **需要排序**：TreeMap、TreeSet **频繁插入删除**：LinkedList **随机访问**：ArrayList **线程安全**：ConcurrentHashMap、Vector

## 扩展知识

这些数据结构的选择直接影响程序性能。在面试中，还需要了解：

- 各种数据结构的时间空间复杂度
- 底层实现原理（如HashMap的hash冲突解决）
- 在不同场景下的适用性
- 线程安全性考虑

掌握这些数据结构的特点和使用场景，是Java开发中的基础技能。