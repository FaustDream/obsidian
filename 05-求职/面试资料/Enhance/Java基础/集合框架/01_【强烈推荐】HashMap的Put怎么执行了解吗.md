# HashMap的Put怎么执行了解吗

HashMap的put方法执行流程是Java面试中的经典问题，我来详细解释一下：

## put方法的核心执行流程

### 1. 计算hash值

```java
// 先计算key的hash值
int hash = hash(key);
```

HashMap会对key的hashCode()进行==二次哈希==，使用`(h = key.hashCode()) ^ (h >>> 16)`，这样做是为了让高位和低位都参与hash计算，减少碰撞。[^1]

### 2. 确定数组索引位置

```java
// 通过hash值和数组长度计算索引
int index = (n - 1) & hash;  // n是数组长度
```

这里用位运算代替取模操作，提高性能。

位运算和取模运算等价性：`hash % 16` 等价于 `hash & 15`。**位运算特性**：`hash & (length-1)` 的结果必然在 `[0, length-1]` 范围内。

### 3. 插入桶元素时处理不同情况

**情况1：待插入的数组桶位置为空（不存在链表节点）**

- 直接创建新链表节点然后赋值给数组桶的索引下标处

**情况2：待插入的数组桶位置已存在链表**
> 相同 hash 值的元素会以链表形式存储在同一个桶中（相同 hash 值会计算到同一个数组索引下标值）

先遍历链表/红黑树，查找相同 key。

```java
// 链表节点数据结构
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;    // 哈希值
    final K key;       // 键
    V value;           // 值
    Node<K,V> next;    // 指向下一个Entry的指针
}
```

- 找到相同 key，则用新 value 替换旧 value。
- 未找到则说明发生 hash（key）冲突，需要处理冲突（在链表末尾或红黑树中插入新节点）。

### ==4. 冲突处理（重点）==

**JDK 1.8之前：**

- 纯链表结构，新元素插入到链表==头部==（头插法）

**JDK 1.8及之后[^2]：**
> HashMap内部使用一个Node数组来存储数据，默认初始容量：16。相同hash值的元素会以链表形式存储在同一个桶中

- 链表长度 ≤ 8：使用链表，新元素插入到链表==尾部==（尾插法）
- 链表长度 > 8 且数组长度 ≥ 64：转换为==红黑树==（**红黑树转回链表的阈值**：红黑树节点数量 ≤ 6）
- 链表长度 > 8 但数组长度 < 64：==优先扩容而不是树化==

### ==5. 数组桶扩容检查==

```java
// threshold: capacity * 0.75
if (++size > threshold) {
    resize();  // 扩容: 2 * currentSize
}
```

==当数组元素个数超过阈值（capacity * loadFactor，默认0.75）时触发扩容。==

## 关键代码示例

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    // 1. 如果table为空，先进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    
    // 2. 计算索引，如果该位置为空，直接插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 3. 该位置已有元素，处理冲突
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;  // key相同，记录该节点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 链表处理
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // 链表长度>=8
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 4. 如果key已存在，替换value
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            return oldValue;
        }
    }
    
    // 5. 检查是否需要扩容
    if (++size > threshold)
        resize();
    return null;
}
```

## 扩展知识点

### 1. 为什么容量必须是2的幂？

- 便于使用位运算 `(n-1) & hash` 快速计算索引
- 扩容时元素要么在原位置，要么在原位置+旧容量的位置

### 2. 扩容时的负载因子为什么是0.75？

- 时间和空间的平衡：太小浪费空间，太大增加hash冲突概率
- 0.75是经过大量测试得出的最优值

### 3. 链表转红黑树的条件

- 链表长度 > 8 **且** 数组长度 ≥ 64
- 红黑树退化为链表的条件是节点数 ≤ 6

### 4. 线程安全问题

- HashMap是线程不安全的
- 并发环境下可能出现死循环（JDK 1.7的头插法）或数据丢失
- 线程安全的替代方案：ConcurrentHashMap、Collections.synchronizedMap()

这个put方法的实现体现了HashMap在性能优化方面的精妙设计，是理解Java集合框架的重要基础。

-----
-----
------

[^1]: 为什么要二次哈希

## 什么是二次哈希

HashMap不是直接使用`key.hashCode()`的返回值，而是对它进行了额外处理：

```java
(h = key.hashCode()) ^ (h >>> 16)
```

## 具体步骤分解

1. `h = key.hashCode()` - 获取key的原始哈希值
2. `h >>> 16` - 将h无符号右移16位
3. `h ^ (h >>> 16)` - 将原始值与右移后的值进行异或运算

## 为什么这样做

**问题背景：** HashMap内部数组的索引是通过 `hash & (length-1)` 计算的。当数组长度较小时，实际上只有哈希值的低位参与了索引计算，高位被忽略了。

**举个例子：**

- 假设HashMap数组长度是16（二进制：10000）
- length-1 = 15（二进制：01111）
- 任何哈希值与01111做AND运算，只有最低4位有效

**解决方案：** 通过`h ^ (h >>> 16)`，让高16位的信息"混合"到低16位中：

```
原始哈希值:  11010110 01001100 10110010 11001010
右移16位:   00000000 00000000 11010110 01001100  
异或结果:   11010110 01001100 01100100 10100110
```

这样，即使最终只使用低位做索引，高位的信息也通过异或运算影响了最终结果，让哈希分布更均匀，减少不同key映射到同一索引的概率（即减少哈希冲突）。

-----
-----
-----

[^2]: 数组长度和链表长度

在HashMap中，**链表长度**和**数组长度**是两个不同的概念：

## 数组长度（桶数量）

- HashMap内部使用一个Node数组来存储数据
- **默认初始容量**：16
- **最大容量**：2^30 (约10亿)
- **扩容规则**：当元素数量超过 `容量 × 负载因子(0.75)` 时，容量翻倍
- **特点**：始终是2的幂次方（16, 32, 64, 128...）

## 链表长度（每个桶中的元素数量）

- 当多个key的哈希值映射到同一个数组索引时，会在该位置形成链表
- **转换为红黑树的阈值**：链表长度 ≥ 8
- **红黑树转回链表的阈值**：红黑树节点数量 ≤ 6
- **理想情况**：每个桶中只有0-1个元素

## 举例说明

```java
HashMap<String, Integer> map = new HashMap<>();
// 数组长度：16（默认）
// 各个桶的链表长度：大部分为0，有数据的桶理想情况下长度为1

// 如果哈希冲突严重，某个桶可能变成：
// 桶[5]: Node1 -> Node2 -> Node3 -> Node4 -> Node5 -> Node6 -> Node7 -> Node8
// 当链表长度达到8时，会转换为红黑树来提高查找效率
```

**总结**：数组长度决定了HashMap的整体容量，而链表长度反映了哈希冲突的程度。好的哈希函数应该让链表长度尽可能短。