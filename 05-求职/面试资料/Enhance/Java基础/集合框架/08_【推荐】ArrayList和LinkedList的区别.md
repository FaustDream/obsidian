# ArrayList和LinkedList的区别

ArrayList和LinkedList是Java中两种重要的List实现，它们在底层数据结构和性能特征上有显著差异：

## 底层数据结构

- **ArrayList**：基于动态数组（Object[]）实现，元素在内存中连续存储。
- **LinkedList**：基于双向链表实现，每个节点包含数据和指向前后节点的引用。

## 性能对比

### 随机访问（get/set）

- **ArrayList**：O(1) - 直接通过索引访问数组元素
- **LinkedList**：O(n) - 需要从头或尾开始遍历到目标位置

### 插入和删除

**ArrayList**：

- 末尾操作：O(1)（平摊时间复杂度）
- 中间/开头操作：O(n)（需要移动元素）

**LinkedList**：

- 任意位置：O(1)（如果已知节点位置）
- 按索引操作：O(n)（需要先找到节点）

## 内存使用

**ArrayList**：

- 内存占用相对较小
- 存在容量浪费（通常预分配50%的额外空间）

**LinkedList**：

- 每个节点需要额外存储两个引用（前驱和后继）
- 内存开销较大，但按需分配

## 适用场景

**ArrayList适合**：

- 频繁随机访问元素
- 读多写少的场景
- 对内存使用敏感的应用

**LinkedList适合**：

- 频繁在中间位置插入/删除
- 实现栈、队列等数据结构
- 不需要随机访问的场景

## 扩展知识

### 线程安全性

两者都不是线程安全的。如需线程安全，可以使用：

- `Collections.synchronizedList()`
- `CopyOnWriteArrayList`（读多写少）
- `Vector`（类似ArrayList但同步）

### 迭代器差异

- ArrayList的Iterator是fail-fast的
- LinkedList还实现了Deque接口，提供了丰富的队列操作方法

### 实际选择建议

在实际开发中，ArrayList使用频率更高（约80-90%的场景），因为大多数应用场景都是读多写少，且随机访问需求较多。只有在确实需要频繁进行中间位置的插入删除操作时，才考虑使用LinkedList。