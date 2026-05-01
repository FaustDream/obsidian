# 自己写一个队列，然后问你如果是阻塞队列还需要什么元素，最后让你实现offer方法



## 1. 基础队列实现

首先实现一个基本的队列：## 2. 阻塞队列需要的额外元素

import java.util.concurrent.atomic.AtomicInteger;

/**
 * 基础队列实现 - 使用链表结构
 */
public class BasicQueue<T> {
    
    // 队列节点
    private static class Node<T> {
        T data;
        Node<T> next;
        
        Node(T data) {
            this.data = data;
        }
    }
    
    private Node<T> head;  // 队头指针
    private Node<T> tail;  // 队尾指针
    private AtomicInteger size = new AtomicInteger(0);  // 队列大小
    
    public BasicQueue() {
        // 初始化一个哨兵节点，简化操作
        head = tail = new Node<>(null);
    }
    
    /**
     * 入队操作
     */
    public boolean offer(T item) {
        if (item == null) {
            throw new NullPointerException("队列不允许null元素");
        }
        
        Node<T> newNode = new Node<>(item);
        tail.next = newNode;
        tail = newNode;
        size.incrementAndGet();
        return true;
    }
    
    /**
     * 出队操作
     */
    public T poll() {
        if (head.next == null) {
            return null;  // 队列为空
        }
        
        Node<T> first = head.next;
        T data = first.data;
        head.next = first.next;
        
        // 如果队列变空，重置tail指针
        if (head.next == null) {
            tail = head;
        }
        
        size.decrementAndGet();
        return data;
    }
    
    /**
     * 查看队头元素但不移除
     */
    public T peek() {
        if (head.next == null) {
            return null;
        }
        return head.next.data;
    }
    
    /**
     * 获取队列大小
     */
    public int size() {
        return size.get();
    }
    
    /**
     * 判断队列是否为空
     */
    public boolean isEmpty() {
        return head.next == null;
    }
}

要实现阻塞队列，需要添加以下核心元素：

**同步机制：**

- `ReentrantLock` - 保证线程安全的互斥锁
- `Condition` - 用于线程间的等待/通知机制

**容量控制：**

- `capacity` - 队列的最大容量
- 容量检查逻辑

**阻塞语义：**

- 当队列满时，入队操作阻塞等待
- 当队列空时，出队操作阻塞等待
- 支持超时和中断

## 3. 完整的阻塞队列实现## 

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 阻塞队列实现
 */
public class BlockingQueue<T> {
    
    // 队列节点
    private static class Node<T> {
        T data;
        Node<T> next;
        
        Node(T data) {
            this.data = data;
        }
    }
    
    private final int capacity;          // 队列容量
    private Node<T> head;                // 队头指针
    private Node<T> tail;                // 队尾指针
    private int size;                    // 当前队列大小
    
    // 同步机制
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();  // 非空条件
    private final Condition notFull = lock.newCondition();   // 非满条件
    
    public BlockingQueue(int capacity) {
        if (capacity <= 0) {
            throw new IllegalArgumentException("容量必须大于0");
        }
        this.capacity = capacity;
        // 初始化哨兵节点
        this.head = this.tail = new Node<>(null);
    }
    
    /**
     * 非阻塞入队 - 队列满时立即返回false
     */
    public boolean offer(T item) {
        if (item == null) {
            throw new NullPointerException("不允许null元素");
        }
        
        lock.lock();
        try {
            // 队列已满，直接返回false
            if (size >= capacity) {
                return false;
            }
            
            // 执行入队操作
            enqueue(item);
            return true;
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 阻塞入队 - 队列满时等待直到有空间
     */
    public void put(T item) throws InterruptedException {
        if (item == null) {
            throw new NullPointerException("不允许null元素");
        }
        
        lock.lock();
        try {
            // 队列满时等待
            while (size >= capacity) {
                notFull.await();
            }
            
            // 执行入队操作
            enqueue(item);
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 超时阻塞入队
     */
    public boolean offer(T item, long timeout, TimeUnit unit) throws InterruptedException {
        if (item == null) {
            throw new NullPointerException("不允许null元素");
        }
        
        long nanos = unit.toNanos(timeout);
        lock.lock();
        try {
            // 队列满时等待指定时间
            while (size >= capacity) {
                if (nanos <= 0) {
                    return false;  // 超时
                }
                nanos = notFull.awaitNanos(nanos);
            }
            
            // 执行入队操作
            enqueue(item);
            return true;
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 执行入队操作的内部方法
     */
    private void enqueue(T item) {
        Node<T> newNode = new Node<>(item);
        tail.next = newNode;
        tail = newNode;
        size++;
        
        // 通知等待的消费者
        notEmpty.signal();
    }
    
    /**
     * 非阻塞出队 - 队列空时立即返回null
     */
    public T poll() {
        lock.lock();
        try {
            if (size == 0) {
                return null;
            }
            
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 阻塞出队 - 队列空时等待直到有元素
     */
    public T take() throws InterruptedException {
        lock.lock();
        try {
            // 队列空时等待
            while (size == 0) {
                notEmpty.await();
            }
            
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 超时阻塞出队
     */
    public T poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        lock.lock();
        try {
            // 队列空时等待指定时间
            while (size == 0) {
                if (nanos <= 0) {
                    return null;  // 超时
                }
                nanos = notEmpty.awaitNanos(nanos);
            }
            
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 执行出队操作的内部方法
     */
    private T dequeue() {
        Node<T> first = head.next;
        T data = first.data;
        head.next = first.next;
        
        // 如果队列变空，重置tail指针
        if (head.next == null) {
            tail = head;
        }
        
        size--;
        
        // 通知等待的生产者
        notFull.signal();
        return data;
    }
    
    /**
     * 查看队头元素但不移除
     */
    public T peek() {
        lock.lock();
        try {
            if (size == 0) {
                return null;
            }
            return head.next.data;
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 获取队列大小
     */
    public int size() {
        lock.lock();
        try {
            return size;
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 判断队列是否为空
     */
    public boolean isEmpty() {
        return size() == 0;
    }
    
    /**
     * 判断队列是否已满
     */
    public boolean isFull() {
        return size() >= capacity;
    }
    
    /**
     * 获取剩余容量
     */
    public int remainingCapacity() {
        lock.lock();
        try {
            return capacity - size;
        } finally {
            lock.unlock();
        }
    }
}

4. 使用示例和测试

import java.util.concurrent.TimeUnit;

/**
 * 阻塞队列测试示例
 */
public class BlockingQueueTest {
    
    public static void main(String[] args) {
        BlockingQueue<String> queue = new BlockingQueue<>(3);
        
        // 测试基本功能
        testBasicFunctions(queue);
        
        // 测试阻塞功能
        testBlockingBehavior(queue);
    }
    
    /**
     * 测试基本功能
     */
    private static void testBasicFunctions(BlockingQueue<String> queue) {
        System.out.println("=== 基本功能测试 ===");
        
        // 测试offer方法
        System.out.println("offer(\"A\"): " + queue.offer("A"));
        System.out.println("offer(\"B\"): " + queue.offer("B"));
        System.out.println("offer(\"C\"): " + queue.offer("C"));
        System.out.println("offer(\"D\"): " + queue.offer("D")); // 应该返回false，队列已满
        
        System.out.println("当前队列大小: " + queue.size());
        System.out.println("队列是否满: " + queue.isFull());
        System.out.println("剩余容量: " + queue.remainingCapacity());
        
        // 测试poll方法
        System.out.println("poll(): " + queue.poll());
        System.out.println("poll(): " + queue.poll());
        System.out.println("peek(): " + queue.peek());
        System.out.println("poll(): " + queue.poll());
        System.out.println("poll(): " + queue.poll()); // 应该返回null，队列为空
        
        System.out.println("队列是否空: " + queue.isEmpty());
        System.out.println();
    }
    
    /**
     * 测试阻塞功能
     */
    private static void testBlockingBehavior(BlockingQueue<String> queue) {
        System.out.println("=== 阻塞功能测试 ===");
        
        // 生产者线程
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = "Item" + i;
                    System.out.println("生产者准备放入: " + item);
                    queue.put(item);  // 阻塞式入队
                    System.out.println("生产者成功放入: " + item);
                    Thread.sleep(1000);  // 模拟生产时间
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // 消费者线程
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    System.out.println("消费者准备取出...");
                    String item = queue.take();  // 阻塞式出队
                    System.out.println("消费者成功取出: " + item);
                    Thread.sleep(1500);  // 模拟消费时间
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // 启动线程
        producer.start();
        consumer.start();
        
        // 等待线程完成
        try {
            producer.join();
            consumer.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("最终队列大小: " + queue.size());
        
        // 测试超时功能
        testTimeoutBehavior(queue);
    }
    
    /**
     * 测试超时功能
     */
    private static void testTimeoutBehavior(BlockingQueue<String> queue) {
        System.out.println("\n=== 超时功能测试 ===");
        
        // 测试超时入队
        try {
            queue.put("X");
            queue.put("Y");
            queue.put("Z");  // 填满队列
            
            long start = System.currentTimeMillis();
            boolean result = queue.offer("Timeout", 2, TimeUnit.SECONDS);
            long end = System.currentTimeMillis();
            
            System.out.println("超时入队结果: " + result);
            System.out.println("等待时间: " + (end - start) + "ms");
            
            // 测试超时出队
            queue.poll(); // 取出一个元素，让队列不为空
            start = System.currentTimeMillis();
            String item = queue.poll(1, TimeUnit.SECONDS);
            end = System.currentTimeMillis();
            
            System.out.println("超时出队结果: " + item);
            System.out.println("等待时间: " + (end - start) + "ms");
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}


5.## 核心知识点总结


### 1. **offer方法的实现要点：**

- **非阻塞特性**：队列满时立即返回false，不等待
- **线程安全**：使用ReentrantLock保证同步
- **容量检查**：先检查容量，再执行入队操作
- **条件通知**：入队成功后通知等待的消费者线程

### 2. **阻塞队列的核心组件：**

- **ReentrantLock**：提供互斥访问
- **Condition对象**：
    - `notEmpty`：队列非空条件，用于消费者等待
    - `notFull`：队列非满条件，用于生产者等待
- **容量控制**：限制队列最大大小
- **阻塞语义**：put/take方法提供阻塞行为

### 3. **扩展知识：**

- **公平性**：可以使用公平锁实现FIFO的线程调度
- **性能优化**：可以考虑使用双锁机制（读写分离）
- **中断处理**：支持线程中断，提高系统响应性
- **JDK实现**：ArrayBlockingQueue、LinkedBlockingQueue的设计思路

这个实现展示了从基础数据结构到并发编程的完整思路，是面试中考察多线程和数据结构知识的经典题目。