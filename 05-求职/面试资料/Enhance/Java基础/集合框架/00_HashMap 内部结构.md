## 具体的存储结构

想象HashMap的内部结构是这样的：

```
数组索引    存储内容
[0]  ──→   null
[1]  ──→   Entry1 ──→ Entry2 ──→ Entry3 ──→ null
[2]  ──→   null  
[3]  ──→   Entry4 ──→ null
[4]  ──→   null
[5]  ──→   Entry5 ──→ Entry6 ──→ null
...
```

## 详细解释

### 1. **数组元素存储的是什么？**

- 数组的每个位置存储的是**第一个Entry的引用**（链表头节点）
- 不是直接存储键值对，而是存储指向Entry链表的指针

### 2. **"串联"是什么意思？**

当多个不同的key经过哈希计算后得到相同的数组索引时，就发生了**哈希冲突**：

```java
// 假设数组长度为16
"name".hashCode() % 16 = 5    // key1映射到索引5
"city".hashCode() % 16 = 5    // key2也映射到索引5！
"age".hashCode() % 16 = 5     // key3也映射到索引5！
```

这三个Entry就会通过链表"串联"在索引5的位置：

```
数组[5] ──→ Entry("name","张三") ──→ Entry("city","北京") ──→ Entry("age",25) ──→ null
```

### 3. **实际的Entry结构**

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;    // 哈希值
    final K key;       // 键
    V value;           // 值
    Node<K,V> next;    // 指向下一个Entry的指针
}
```

### 4. **查找过程**

当我们查找`get("city")`时：

1. 计算`"city"`的哈希值，得到索引5
2. 从`数组[5]`开始，沿着链表逐个比较：
    - 比较Entry("name") → key不匹配，继续
    - 比较Entry("city") → key匹配！返回"北京"

这就是为什么说Entry"串联"在数组索引位置 - 它们通过链表的next指针连接起来，挂在同一个数组位置上。