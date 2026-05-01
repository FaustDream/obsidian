# run和start区别

在Java线程编程中，`run()`和`start()`是两个容易混淆但功能完全不同的方法：

## 核心区别

**start()方法**：

- 启动一个新的线程
- 由JVM调用底层系统创建新线程
- 新线程会自动调用`run()`方法
- 只能调用一次，重复调用会抛出`IllegalThreadStateException`

**run()方法**：

- 线程的执行体，包含线程要执行的代码
- 直接调用时在当前线程中执行，不会创建新线程
- 可以多次调用

## 代码示例

```java
public class ThreadDemo extends Thread {
    @Override
    public void run() {
        System.out.println("当前线程: " + Thread.currentThread().getName());
        for (int i = 0; i < 3; i++) {
            System.out.println("执行中: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        ThreadDemo thread = new ThreadDemo();
        
        // 使用start() - 创建新线程
        thread.start();
        // 输出: 当前线程: Thread-0
        
        // 直接调用run() - 在main线程中执行
        ThreadDemo thread2 = new ThreadDemo();
        thread2.run();
        // 输出: 当前线程: main
    }
}
```

## 底层原理

**start()方法的执行流程**：

1. 检查线程状态，确保未启动
2. 将线程加入线程组
3. 调用native方法`start0()`
4. 操作系统创建新线程
5. 新线程调用`run()`方法

**线程状态变化**：

- 调用`start()`：NEW → RUNNABLE
- 直接调用`run()`：状态不变，仍为NEW

## 扩展知识点

### 1. 线程安全问题

```java
// 错误示例 - 重复调用start()
Thread t = new Thread(() -> System.out.println("Hello"));
t.start();
t.start(); // 抛出IllegalThreadStateException
```

### 2. 实现Runnable接口的情况

```java
public class RunnableDemo implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable: " + Thread.currentThread().getName());
    }
}

// 使用方式
RunnableDemo runnable = new RunnableDemo();
Thread thread = new Thread(runnable);
thread.start(); // 正确 - 创建新线程
runnable.run(); // 在当前线程执行
```

### 3. 线程池中的应用

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
executor.submit(() -> {
    // 这里的代码实际上是在run()方法中执行
    System.out.println("线程池中的线程: " + Thread.currentThread().getName());
});
```

### 4. 性能考虑

- `start()`涉及系统调用，开销较大
- `run()`只是普通方法调用，开销小
- 频繁创建线程建议使用线程池

**面试要点**：理解这两个方法的区别不仅是基础语法问题，更反映了对Java线程模型和JVM底层机制的理解程度。