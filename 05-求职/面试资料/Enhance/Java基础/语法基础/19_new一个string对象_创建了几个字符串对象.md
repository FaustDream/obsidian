# new一个string对象，创建了几个字符串对象

## 情况分析

### 1. `new String("hello")` 的情况

```java
String str = new String("hello");
```

**创建了2个字符串对象：**

- 一个在**字符串常量池**中：`"hello"`
- 一个在**堆内存**中：通过new关键字创建的String对象

### 2. `new String()` 的情况

```java
String str = new String();
```

**创建了1个字符串对象：**

- 只在堆内存中创建一个空字符串对象

## 深入理解

### 字符串常量池机制

```java
// 第一次出现"hello"时
String str1 = new String("hello"); // 创建2个对象

// 第二次出现"hello"时
String str2 = new String("hello"); // 只创建1个对象（堆中）
// 因为常量池中已经存在"hello"了
```

### 验证代码

```java
public class StringTest {
    public static void main(String[] args) {
        String str1 = new String("hello");
        String str2 = "hello";
        String str3 = new String("hello");
        
        System.out.println(str1 == str2);        // false
        System.out.println(str1 == str3);        // false
        System.out.println(str2 == "hello");     // true
        System.out.println(str1.equals(str2));   // true
    }
}
```

## 扩展知识点

### 1. 字符串拼接的对象创建

```java
String str = new String("hel") + new String("lo");
// 实际创建的对象数量会更多，涉及StringBuilder等
```

### 2. intern()方法的作用

```java
String str1 = new String("hello");
String str2 = str1.intern();
String str3 = "hello";
System.out.println(str2 == str3); // true
```

### 3. 编译时优化

```java
String str1 = "hel" + "lo";        // 编译时优化，等同于"hello"
String str2 = new String("hel" + "lo"); // 仍然创建2个对象
```

## 内存分布图解

```
堆内存 Heap:
┌─────────────────┐
│ String对象      │ ← new String()创建
│ value: 指向常量池│
└─────────────────┘

字符串常量池 (元空间):
┌─────────────────┐
│ "hello"         │ ← 字面量创建
└─────────────────┘
```

## 面试加分回答要点

1. **明确区分**堆内存和字符串常量池
2. **强调**字符串常量池的复用机制
3. **提到**JDK版本差异（JDK7后常量池移到堆中）
4. **展示**对内存管理的理解
5. **能够**手写验证代码

这个问题考查的是对Java内存模型和字符串机制的深度理解，是基础但非常重要的知识点。