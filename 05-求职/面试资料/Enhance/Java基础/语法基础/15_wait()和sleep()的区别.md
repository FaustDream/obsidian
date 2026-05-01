# wait()和sleep()的区别

**wait()和sleep()的核心区别**

这两个方法最本质的区别在于**锁的处理方式**：

## 1. 锁的释放

- **wait()**: 会**释放**当前对象的锁，让其他线程可以获取锁
- **sleep()**: **不释放**任何锁，继续持有已获取的锁

## 2. 所属类和调用方式

- **wait()**: Object类的方法，==必须在synchronized块中调用==
- **sleep()**: Thread类的静态方法，可以在任何地方调用

## 3. 唤醒机制

- **wait()**: 需要其他线程调用notify()或notifyAll()来唤醒
- **sleep()**: 时间到了自动唤醒，或被interrupt()中断

## 4. 实际应用场景

```java
// wait()的典型用法 - 生产者消费者模式
public class Producer {
    private Object lock = new Object();
    private boolean ready = false;
    
    public void consume() throws InterruptedException {
        synchronized(lock) {
            while(!ready) {
                lock.wait(); // 释放锁，等待生产者通知
            }
            // 消费逻辑
        }
    }
    
    public void produce() {
        synchronized(lock) {
            ready = true;
            lock.notify(); // 唤醒等待的消费者
        }
    }
}

// sleep()的典型用法 - 定时任务
public class Timer {
    public void periodicTask() {
        while(true) {
            doSomething();
            try {
                Thread.sleep(1000); // 暂停1秒，不释放锁
            } catch(InterruptedException e) {
                break;
            }
        }
    }
}
```

## 5. 相关扩展知识

**为什么wait()必须在synchronized中调用？**

- 防止竞态条件：==确保wait()和notify()的调用是原子性的==
- 避免虚假唤醒问题

**替代方案：**

- wait()/notify() → **CountDownLatch、Semaphore、BlockingQueue**
- sleep() → **ScheduledExecutorService**

**面试延伸点：**

- **虚假唤醒**(spurious wakeup)的处理
- **锁升级**过程中wait()的行为
- **线程池**中sleep()对工作线程的影响
- **JVM层面**：wait()涉及monitor机制，sleep()是系统调用

记住：**wait()是==线程间协作==的工具，sleep()是线程自身的休眠工具**。