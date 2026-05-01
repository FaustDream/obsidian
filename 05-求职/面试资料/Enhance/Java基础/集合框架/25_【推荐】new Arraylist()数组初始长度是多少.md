在Java中，`new ArrayList()`创建的数组列表**初始长度是0**，但其**内部数组容量是10**。

让我详细解释一下：

## 基本概念区分

**长度（size）** vs **容量（capacity）**：

- **长度（size）**：当前ArrayList中实际存储的元素个数
- **容量（capacity）**：内部数组能够容纳的元素总数

## ArrayList的初始化过程

```java
// 使用无参构造器
ArrayList<String> list = new ArrayList<>();
System.out.println(list.size()); // 输出：0（长度为0）
```

## 源码分析

在ArrayList的源码中：

```java
public class ArrayList<E> extends AbstractList<E> {
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    // 无参构造器
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
}
```

## 容量扩展机制

当第一次添加元素时，ArrayList会将容量扩展到默认容量10：

```java
ArrayList<String> list = new ArrayList<>();
System.out.println(list.size()); // 0

list.add("第一个元素");
System.out.println(list.size()); // 1
// 此时内部数组容量变为10
```

## 相关知识扩展

### 1. 其他构造器的初始容量

```java
// 指定初始容量
ArrayList<String> list1 = new ArrayList<>(15); // 容量为15

// 基于其他集合创建
ArrayList<String> list2 = new ArrayList<>(Arrays.asList("a", "b")); // 容量为2
```

### 2. 扩容策略

- 当容量不足时，新容量 = 旧容量 + (旧容量 >> 1)
- 即每次扩容约1.5倍

### 3. 性能优化建议

如果预知元素数量，建议指定初始容量避免频繁扩容：

```java
// 如果知道要存储100个元素
ArrayList<String> list = new ArrayList<>(100);
```

这样可以减少扩容操作，提高性能。