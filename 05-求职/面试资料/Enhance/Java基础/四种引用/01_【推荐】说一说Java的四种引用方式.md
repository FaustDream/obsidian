# 说一说Java的四种引用方式

Java中有四种引用类型，它们的强度依次递减，分别是：强引用、软引用、弱引用和虚引用。

## 1. 强引用（Strong Reference）

**定义**：这是最常见的引用类型，也是默认的引用方式。

```java
Object obj = new Object(); // 强引用
String str = "Hello World"; // 强引用
```

**特点**：

- 只要强引用存在，垃圾收集器永远不会回收被引用的对象
- 即使内存不足抛出OutOfMemoryError，也不会回收强引用对象
- 可以通过将引用赋值为null来中断强引用

## 2. 软引用（Soft Reference）

**定义**：用SoftReference类实现，在内存不足时才会被垃圾收集器回收。

```java
Object obj = new Object(); // 强引用
SoftReference<Object> softRef = new SoftReference<>(obj); // 弱引用obj
obj = null; // 断开强引用

// 获取软引用对象
Object retrievedObj = softRef.get(); // 可能返回null
```

**特点**：

- 内存==充足==时不会被回收
- 内存==不足==时会被回收，避免OutOfMemoryError
- 适用于实现内存敏感的缓存

**应用场景**：图片缓存、网页缓存等

## 3. 弱引用（Weak Reference）

**定义**：用WeakReference类实现，在下次垃圾收集时就会被回收。

```java
Object obj = new Object();
WeakReference<Object> weakRef = new WeakReference<>(obj);
obj = null; // 断开强引用

// 下次GC后，weakRef.get()可能返回null
Object retrievedObj = weakRef.get();
```

**特点**：

- 不能阻止对象被垃圾收集
- 一旦发生GC，弱引用对象就可能被回收
- 比软引用更容易被回收

**应用场景**：

- WeakHashMap的实现
- 监听器模式，避免内存泄漏
- ==ThreadLocalMap中的key==

## 4. 虚引用（Phantom Reference）

**定义**：用PhantomReference类实现，最弱的引用类型。

```java
Object obj = new Object();
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue);
obj = null;

// phantomRef.get()始终返回null
```

**特点**：

- get()方法始终返回null
- 必须与ReferenceQueue联合使用
- 主要用于跟踪对象被垃圾收集的状态
- 对象回收前会收到通知

**应用场景**：

- 监控对象回收
- 堆外内存管理
- ==资源清理工作==

## 引用强度对比

```
强引用 > 软引用 > 弱引用 > 虚引用
```

## 实际应用示例

```java
public class ReferenceExample {
    public static void main(String[] args) {
        // 创建对象
        Object obj = new Object();
        
        // 创建不同类型的引用
        SoftReference<Object> softRef = new SoftReference<>(obj);
        WeakReference<Object> weakRef = new WeakReference<>(obj);
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue);
        
        // 断开强引用
        obj = null;
        
        // 手动触发GC
        System.gc();
        
        // 检查引用状态
        System.out.println("Soft: " + (softRef.get() != null));
        System.out.println("Weak: " + (weakRef.get() != null));
        System.out.println("Phantom: " + (phantomRef.get() != null)); // 始终为null
    }
}
```

## 扩展知识

**ReferenceQueue的作用**：

- 当引用对象被回收时，引用本身会被加入到ReferenceQueue中
- 可以通过检查队列来得知哪些对象已被回收
- 常用于资源清理工作

**ThreadLocal与弱引用**：

- ThreadLocalMap使用弱引用作为key
- 避免了线程长时间运行时的内存泄漏问题
- 但value仍可能造成内存泄漏，需要手动remove

这四种引用类型为Java提供了灵活的内存管理机制，帮助开发者在不同场景下实现合适的内存使用策略。