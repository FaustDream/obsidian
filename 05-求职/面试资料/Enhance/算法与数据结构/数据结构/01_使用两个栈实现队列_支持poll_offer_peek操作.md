# 使用两个栈实现队列，支持poll，offer，peek操作

我来详细解释如何使用两个栈实现队列，并提供完整的代码实现。

## 核心思想

队列是先进先出（FIFO），而栈是后进先出（LIFO）。使用两个栈可以巧妙地实现队列的特性：

- **栈1（inStack）**：负责接收新元素
- **栈2（outStack）**：负责输出元素

## 完整代码实现## 关键知识点解析

### 1. **工作原理**

- **入队（offer）**：直接将元素压入 `inStack`
- **出队（poll/peek）**：
    - 如果 `outStack` 不为空，直接从 `outStack` 弹出/查看
    - 如果 `outStack` 为空，将 `inStack` 中所有元素转移到 `outStack`，然后操作

### 2. **时间复杂度分析**

- **offer**: O(1) - 直接压栈
- **poll/peek**: 摊还 O(1) - 虽然转移操作是 O(n)，但每个元素最多被转移一次

### 3. **空间复杂度**

- O(n) - 需要存储 n 个元素

## 扩展知识

### 面试中的常见变形题

1. **用队列实现栈**
2. **设计最小栈**
3. **用数组实现环形队列**
4. **实现双端队列（Deque）**

### 优化思路

1. **懒惰转移**：只有在需要时才转移元素
2. **批量操作**：可以考虑批量转移以提高效率
3. **容量限制**：可以添加容量检查

### 实际应用场景

1. **消息队列系统**
2. **任务调度器**
3. **缓冲区实现**
4. **BFS算法中的队列**

这种实现方式既满足了队列的基本要求，又充分利用了栈的特性，是一个经典的数据结构设计问题。

import java.util.Stack;

/**
 * 使用两个栈实现队列
 * 支持 offer(入队)、poll(出队)、peek(查看队首元素) 操作
 */
public class QueueWithTwoStacks<T> {
    private Stack<T> inStack;   // 用于入队操作
    private Stack<T> outStack;  // 用于出队操作
    
    public QueueWithTwoStacks() {
        inStack = new Stack<>();
        outStack = new Stack<>();
    }
    
    /**
     * 入队操作 - 将元素添加到队尾
     * 时间复杂度：O(1)
     */
    public void offer(T element) {
        inStack.push(element);
    }
    
    /**
     * 出队操作 - 移除并返回队首元素
     * 时间复杂度：摊还 O(1)
     */
    public T poll() {
        if (isEmpty()) {
            return null; // 或者抛出异常
        }
        
        // 如果outStack为空，将inStack中的所有元素转移到outStack
        if (outStack.isEmpty()) {
            transferElements();
        }
        
        return outStack.pop();
    }
    
    /**
     * 查看队首元素但不移除
     * 时间复杂度：摊还 O(1)
     */
    public T peek() {
        if (isEmpty()) {
            return null; // 或者抛出异常
        }
        
        // 如果outStack为空，将inStack中的所有元素转移到outStack
        if (outStack.isEmpty()) {
            transferElements();
        }
        
        return outStack.peek();
    }
    
    /**
     * 检查队列是否为空
     */
    public boolean isEmpty() {
        return inStack.isEmpty() && outStack.isEmpty();
    }
    
    /**
     * 获取队列大小
     */
    public int size() {
        return inStack.size() + outStack.size();
    }
    
    /**
     * 将inStack中的所有元素转移到outStack
     * 这个操作使得最先进入inStack的元素位于outStack的顶部
     */
    private void transferElements() {
        while (!inStack.isEmpty()) {
            outStack.push(inStack.pop());
        }
    }
    
    /**
     * 测试方法
     */
    public static void main(String[] args) {
        QueueWithTwoStacks<Integer> queue = new QueueWithTwoStacks<>();
        
        // 测试入队
        queue.offer(1);
        queue.offer(2);
        queue.offer(3);
        
        System.out.println("队列大小: " + queue.size()); // 输出: 3
        System.out.println("队首元素: " + queue.peek()); // 输出: 1
        
        // 测试出队
        System.out.println("出队: " + queue.poll()); // 输出: 1
        System.out.println("出队: " + queue.poll()); // 输出: 2
        
        // 再次入队
        queue.offer(4);
        
        System.out.println("队首元素: " + queue.peek()); // 输出: 3
        System.out.println("出队: " + queue.poll()); // 输出: 3
        System.out.println("出队: " + queue.poll()); // 输出: 4
        
        System.out.println("队列是否为空: " + queue.isEmpty()); // 输出: true
    }
}

/**
 * 线程安全版本（可选）
 */
class ThreadSafeQueueWithTwoStacks<T> {
    private Stack<T> inStack;
    private Stack<T> outStack;
    private final Object lock = new Object();
    
    public ThreadSafeQueueWithTwoStacks() {
        inStack = new Stack<>();
        outStack = new Stack<>();
    }
    
    public void offer(T element) {
        synchronized (lock) {
            inStack.push(element);
        }
    }
    
    public T poll() {
        synchronized (lock) {
            if (isEmpty()) {
                return null;
            }
            
            if (outStack.isEmpty()) {
                transferElements();
            }
            
            return outStack.pop();
        }
    }
    
    public T peek() {
        synchronized (lock) {
            if (isEmpty()) {
                return null;
            }
            
            if (outStack.isEmpty()) {
                transferElements();
            }
            
            return outStack.peek();
        }
    }
    
    public boolean isEmpty() {
        synchronized (lock) {
            return inStack.isEmpty() && outStack.isEmpty();
        }
    }
    
    public int size() {
        synchronized (lock) {
            return inStack.size() + outStack.size();
        }
    }
    
    private void transferElements() {
        while (!inStack.isEmpty()) {
            outStack.push(inStack.pop());
        }
    }
}