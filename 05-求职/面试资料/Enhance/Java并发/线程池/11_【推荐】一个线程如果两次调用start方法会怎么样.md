# 一个线程如果两次调用start方法会怎么样

当一个线程两次调用start()方法时，会抛出`IllegalThreadStateException`异常。

## 具体分析

**第一次调用start():**

- 线程状态从NEW变为RUNNABLE
- 线程开始执行run()方法
- 一切正常

**第二次调用start():**

- 由于线程状态已经不是NEW（可能是RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED中的任意一种）
- Thread类的start()方法会检查线程状态
- 发现状态不是NEW，直接抛出IllegalThreadStateException

## 源码分析

```java
public synchronized void start() {
    // 检查线程状态，如果不是NEW状态就抛异常
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
    
    group.add(this);
    boolean started = false;
    try {
        start0(); // 调用native方法
        started = true;
    } finally {
        // 异常处理逻辑
    }
}
```

## 示例代码

```java
public class ThreadStartTest {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("线程执行中...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        thread.start(); // 第一次调用，正常执行
        
        try {
            thread.start(); // 第二次调用，抛异常
        } catch (IllegalThreadStateException e) {
            System.out.println("捕获异常: " + e.getMessage());
        }
    }
}
```

## 相关知识拓展

**线程状态转换图:**

- NEW → RUNNABLE (调用start())
- RUNNABLE → TERMINATED (执行完成)
- 一旦离开NEW状态，就无法再次调用start()

**如果想重复执行相同任务:**

```java
// 错误做法
Thread thread = new Thread(task);
thread.start();
thread.start(); // 异常！

// 正确做法
Thread thread1 = new Thread(task);
Thread thread2 = new Thread(task);
thread1.start();
thread2.start(); // 创建新线程对象
```

**线程池的优势:** 正是因为线程不能重复启动，所以在需要频繁创建线程的场景下，线程池通过复用线程对象来提高性能，避免了重复创建和销毁线程的开销。

这个机制确保了线程的生命周期管理的严格性，防止了线程状态的混乱。