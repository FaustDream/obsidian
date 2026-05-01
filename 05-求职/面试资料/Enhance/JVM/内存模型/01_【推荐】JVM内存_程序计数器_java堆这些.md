# JVM内存（程序计数器，java堆这些）

我来详细解释一下JVM内存结构，这是Java面试中的核心知识点。

## JVM内存结构概述

JVM内存主要分为以下几个区域：

### 1. 程序计数器（Program Counter Register）

- **作用**：记录当前线程正在执行的字节码指令地址
- **特点**：
    - 线程私有，每个线程都有独立的程序计数器
    - 占用内存很小，可以忽略不计
    - 唯一不会发生OutOfMemoryError的内存区域
    - 执行Native方法时，程序计数器值为空（Undefined）

### 2. Java虚拟机栈（Java Virtual Machine Stack）

- **作用**：存储方法调用的局部变量、操作数栈、动态链接、方法出口等信息
- **特点**：
    - 线程私有，生命周期与线程相同
    - 每个方法调用都会创建一个栈帧（Stack Frame）
    - 可能抛出StackOverflowError和OutOfMemoryError

### 3. 本地方法栈（Native Method Stack）

- **作用**：为Native方法服务
- **特点**：
    - 线程私有
    - 与虚拟机栈作用相似，但服务于Native方法
    - 也可能抛出StackOverflowError和OutOfMemoryError

### 4. Java堆（Java Heap）

- **作用**：存储对象实例和数组
- **特点**：
    - 线程共享，JVM启动时创建
    - 垃圾收集器管理的主要区域
    - 可以分为新生代和老年代
    - 容易发生OutOfMemoryError

**堆的分代结构**：

```
Java堆
├── 新生代（Young Generation）
│   ├── Eden区
│   ├── Survivor 0区
│   └── Survivor 1区
└── 老年代（Old Generation）
```

### 5. 方法区（Method Area）

- **作用**：存储类信息、常量、静态变量、即时编译后的代码等
- **特点**：
    - 线程共享
    - 也被称为"永久代"（JDK 8之前）
    - JDK 8后改为元空间（Metaspace），使用直接内存

### 6. 运行时常量池（Runtime Constant Pool）

- **作用**：存放编译期生成的各种字面量和符号引用
- **特点**：
    - 属于方法区的一部分
    - 具有动态性，可以在运行期间将新的常量放入池中

## 内存分配示例

```java
public class MemoryExample {
    // 存储在方法区
    private static String staticVar = "static";
    
    public void method() {
        // 局部变量存储在虚拟机栈
        int localVar = 10;
        
        // 对象存储在堆中，引用存储在栈中
        String str = new String("hello");
        
        // 数组对象存储在堆中
        int[] array = new int[100];
    }
}
```

## 常见面试扩展问题

### 1. 堆和栈的区别

- **存储内容**：栈存储方法调用和局部变量，堆存储对象实例
- **生命周期**：栈随方法调用结束而销毁，堆中对象由GC管理
- **线程安全**：栈线程私有天然线程安全，堆线程共享需要同步
- **性能**：栈分配和回收速度快，堆相对较慢

### 2. 内存溢出场景

- **堆溢出**：创建大量对象且无法被GC回收
- **栈溢出**：递归调用过深或方法调用层次过多
- **方法区溢出**：动态生成大量类

### 3. JDK版本差异

- **JDK 7**：字符串常量池从方法区移到堆中
- **JDK 8**：永久代被元空间取代，使用直接内存
- **JDK 11+**：进一步优化内存管理和GC算法

这些知识点不仅要理解概念，更要能够结合实际开发场景来分析内存使用情况，这样在面试中才能展现出深度理解。