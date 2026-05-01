# ArrayList的扩容机制

ArrayList的扩容机制是Java面试中的经典问题，我来详细解释一下：

## 核心扩容机制

ArrayList的扩容采用**1.5倍扩容策略**：

- 当数组容量不足时，新容量 = 原容量 + 原容量 >> 1
- 即新容量是原容量的1.5倍

## 源码分析

```java
// 核心扩容方法
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 新容量 = 老容量 + 老容量右移1位（相当于除以2）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    
    // 如果新容量还不够，直接使用所需容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
        
    // 检查是否超过最大数组长度
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
        
    // 创建新数组并复制元素
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## 触发扩容的时机

1. **add()方法**：添加元素时容量不足
2. **addAll()方法**：批量添加元素时容量不足
3. **ensureCapacity()方法**：手动触发扩容

## 扩容过程详解

假设当前容量为10的ArrayList：

1. 当添加第11个元素时触发扩容
2. 新容量计算：10 + (10 >> 1) = 10 + 5 = 15
3. 创建长度为15的新数组
4. 将原数组元素复制到新数组
5. 更新引用指向新数组

## 性能考虑

**时间复杂度**：

- 扩容操作：O(n)，需要复制所有元素
- 均摊时间复杂度：O(1)

**空间复杂度**：

- 临时需要2倍的内存空间（新旧数组同时存在）

## 优化建议

```java
// 1. 预估容量，避免频繁扩容
ArrayList<String> list = new ArrayList<>(1000);

// 2. 使用ensureCapacity手动扩容
list.ensureCapacity(2000);

// 3. 批量操作时一次性添加
list.addAll(Arrays.asList("a", "b", "c"));
```

## 相关知识点拓展

### 1. 与其他集合的对比

- **Vector**：2倍扩容，线程安全但性能较差
- **LinkedList**：不需要扩容，但随机访问性能差
- **HashMap**：2倍扩容，负载因子0.75

### 2. 内存泄漏问题

```java
// 大量删除元素后，建议手动缩容
list.trimToSize(); // 将容量调整为实际元素个数
```

### 3. 线程安全问题

ArrayList扩容不是线程安全的，多线程环境下可能出现：

- 数组越界异常
- 数据丢失
- 死循环

**解决方案**：

```java
// 1. 使用Collections.synchronizedList()
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// 2. 使用CopyOnWriteArrayList
List<String> cowList = new CopyOnWriteArrayList<>();

// 3. 使用Vector（不推荐）
Vector<String> vector = new Vector<>();
```

### 4. 序列化相关

ArrayList实现了Serializable接口，但elementData数组被transient修饰，采用自定义序列化方式只序列化有效元素，节省空间。

这个扩容机制的设计既保证了性能（均摊O(1)），又避免了内存浪费（相比2倍扩容），是ArrayList高效性能的重要保障。