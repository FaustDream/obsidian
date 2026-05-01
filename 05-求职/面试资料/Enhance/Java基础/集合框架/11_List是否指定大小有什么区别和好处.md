# List是否指定大小有什么区别和好处

在Java中，List是否指定初始大小主要体现在ArrayList的构造方法上，这个选择会显著影响性能和内存使用。让我详细解释一下：

## 不指定大小（默认构造）

```java
List<String> list = new ArrayList<>(); // 默认初始容量为10
```

## 指定大小

```java
List<String> list = new ArrayList<>(100); // 指定初始容量为100
```

## 主要区别和好处

### 1. 性能差异

**不指定大小的问题：**

- 当元素数量超过当前容量时，ArrayList需要进行**动态扩容**
- 扩容过程：创建新数组（通常是原容量的1.5倍）→ 复制所有元素 → 替换原数组
- 频繁扩容会导致大量的数组复制操作，时间复杂度为O(n)

**指定合适大小的好处：**

- 避免或减少扩容操作
- 元素添加操作保持O(1)时间复杂度
- 整体性能提升明显

### 2. 内存使用效率

```java
// 场景：需要存储1000个元素
List<String> list1 = new ArrayList<>();        // 会经历多次扩容
List<String> list2 = new ArrayList<>(1000);    // 一次性分配足够空间
```

**指定大小的内存优势：**

- 减少内存碎片
- 避免临时创建过大的中间数组
- 更好的内存局部性

### 3. 实际应用场景

**适合指定大小的情况：**

```java
// 1. 已知数据量大小
List<User> users = new ArrayList<>(database.getUserCount());

// 2. 批量数据处理
List<String> lines = new ArrayList<>(Files.readAllLines(path).size());

// 3. 集合转换
Set<String> set = getDataSet();
List<String> list = new ArrayList<>(set.size());
```

**不需要指定大小的情况：**

- 数据量很小（<100个元素）
- 无法预估最终大小
- 临时使用的集合

## 性能测试示例

```java
// 测试添加100万个元素的性能差异
long start = System.currentTimeMillis();

// 不指定大小
List<Integer> list1 = new ArrayList<>();
for(int i = 0; i < 1000000; i++) {
    list1.add(i);
}
System.out.println("不指定大小耗时：" + (System.currentTimeMillis() - start) + "ms");

start = System.currentTimeMillis();
// 指定大小
List<Integer> list2 = new ArrayList<>(1000000);
for(int i = 0; i < 1000000; i++) {
    list2.add(i);
}
System.out.println("指定大小耗时：" + (System.currentTimeMillis() - start) + "ms");
```

## 扩展知识点

### 1. 其他Collection实现

- **HashMap**: `new HashMap<>(expectedSize / 0.75f + 1)` 避免rehash
- **HashSet**: 同HashMap，底层就是HashMap实现
- **LinkedList**: 不支持初始容量，因为是链表结构

### 2. 容量计算策略

```java
// 如果知道确切大小
List<String> list = new ArrayList<>(exactSize);

// 如果是估算大小，建议预留一些空间
List<String> list = new ArrayList<>(estimatedSize + estimatedSize / 4);
```

### 3. 相关面试延伸

- ArrayList扩容机制源码分析
- 为什么扩容因子是1.5而不是2
- Vector和ArrayList的区别
- 何时选择LinkedList vs ArrayList

**总结：** 指定合适的初始大小是一个重要的性能优化手段，特别是在处理大量数据时。这体现了"空间换时间"的优化思想，也是高级Java开发者应该掌握的最佳实践。