# Java有哪些对象的引用类型

Java中有四种不同的对象引用类型，它们的强度依次递减：

## 1. 强引用（Strong Reference）

**最常见的引用类型**，平时我们创建的对象都是强引用：

```java
Object obj = new Object(); // 强引用
String str = "Hello";      // 强引用
```

- 只要强引用存在，垃圾收集器永远不会回收被引用的对象
- 即使内存不足，JVM宁愿抛出OutOfMemoryError也不会回收强引用对象
- 通过将引用设置为null来断开强引用

## 2. 软引用（Soft Reference）

**内存敏感的缓存场景**，当内存不足时会被回收：

```java
SoftReference<Object> softRef = new SoftReference<>(new Object());
Object obj = softRef.get(); // 可能返回null
```

- 只有在内存不足时才会被垃圾收集器回收
- 适合实现内存敏感的高速缓存
- 可以配合引用队列使用来跟踪被回收的对象

## 3. 弱引用（Weak Reference）

**生命周期更短**，下次GC时就会被回收：

```java
WeakReference<Object> weakRef = new WeakReference<>(new Object());
Object obj = weakRef.get(); // 可能返回null
```

- 无论内存是否充足，下次垃圾收集时都会被回收
- 常用于解决内存泄漏问题
- ThreadLocal的实现就使用了弱引用

## 4. 虚引用（Phantom Reference）

**最弱的引用类型**，主要用于跟踪对象的回收：

```java
PhantomReference<Object> phantomRef = new PhantomReference<>(new Object(), queue);
Object obj = phantomRef.get(); // 总是返回null
```

- get()方法总是返回null
- 必须配合引用队列使用
- 主要用于跟踪对象被回收的状态，做清理工作

## 实际应用场景

**软引用** - 图片缓存、数据缓存

```java
Map<String, SoftReference<Bitmap>> imageCache = new HashMap<>();
```

**弱引用** - 监听器模式、ThreadLocal

```java
WeakHashMap<Key, Value> map = new WeakHashMap<>();
```

**虚引用** - 对象回收监控、堆外内存清理

```java
// DirectByteBuffer的清理就使用了虚引用
```

## 引用队列（Reference Queue）

软引用、弱引用和虚引用都可以配合引用队列使用，当引用的对象被回收时，引用本身会被放入队列中，便于我们做后续的清理工作。

这种设计让Java的内存管理更加灵活，可以根据不同的使用场景选择合适的引用类型。