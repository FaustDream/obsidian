# for循环里拼接字符串会有什么问题

在for循环中拼接字符串会产生严重的性能问题，主要原因是Java中String对象的不可变性。

## 主要问题

### 1. 性能问题

```java
// 问题代码示例
String result = "";
for (int i = 0; i < 1000; i++) {
    result += "Hello " + i;  // 每次都创建新的String对象
}
```

每次执行 `+=` 操作时，JVM会：

- 创建一个新的String对象
- 将原字符串内容复制到新对象
- 将新内容追加到新对象
- 丢弃原String对象

这导致时间复杂度为O(n²)，空间上也会产生大量垃圾对象。

### 2. 内存消耗

- 每次拼接都会在堆内存中创建新对象
- 产生大量临时对象，增加GC压力
- 内存使用效率低下

## 解决方案

### 1. 使用StringBuilder（推荐）

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("Hello ").append(i);
}
String result = sb.toString();
```

### 2. 使用StringBuffer（线程安全）

```java
StringBuffer sb = new StringBuffer();
for (int i = 0; i < 1000; i++) {
    sb.append("Hello ").append(i);
}
String result = sb.toString();
```

### 3. 预估容量优化

```java
StringBuilder sb = new StringBuilder(1000 * 10); // 预估容量
for (int i = 0; i < 1000; i++) {
    sb.append("Hello ").append(i);
}
```

## 性能对比

- **String拼接**: O(n²) 时间复杂度
- **StringBuilder**: O(n) 时间复杂度
- **内存使用**: StringBuilder减少90%以上的对象创建

## 扩展知识

### 编译器优化

现代Java编译器会将简单的字符串拼接优化为StringBuilder：

```java
String s = "a" + "b" + "c";  // 编译器优化为StringBuilder
```

==但在循环中无法进行此优化。==

### JDK 9+ 改进

JDK 9引入了StringConcatFactory，通过invokedynamic指令优化字符串拼接，但循环场景仍建议使用StringBuilder。

### 最佳实践

1. 循环中大量字符串拼接必须使用StringBuilder
2. 能预估最终长度时设置初始容量
3. 多线程环境考虑使用StringBuffer
4. 简单的少量拼接可以直接使用+号

这个问题体现了对Java内存模型和字符串处理机制的深度理解，是面试中的经典问题。