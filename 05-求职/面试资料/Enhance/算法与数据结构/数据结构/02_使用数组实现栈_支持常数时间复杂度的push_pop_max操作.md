# 使用数组实现栈，支持常数时间复杂度的push，pop，max操作

我来为您实现一个支持常数时间复杂度的push、pop、max操作的栈。

核心思想是使用**辅助栈**来维护最大值信息。## 实现要点解析

### 1. 核心设计思想

- **双栈结构**：使用两个数组分别存储数据和最大值
- **dataStack**：存储实际的栈数据
- **maxStack**：存储对应位置的最大值信息

### 2. 时间复杂度分析

**push操作 - O(1)**：

- 直接在数组末尾添加元素
- 通过比较决定是否更新最大值栈

**pop操作 - O(1)**：

- 直接从数组末尾移除元素
- 如果移除的是最大值，同时更新最大值栈

**max操作 - O(1)**：

- 直接返回最大值栈的栈顶元素

### 3. 空间复杂度

- **空间复杂度**：O(n)，其中n是栈中元素个数
- 最坏情况下，如果所有元素都是递增的，两个栈都会存储所有元素

### 4. 关键优化点

**最大值栈的优化**：

- 只有当新元素 **大于等于** 当前最大值时，才将其推入最大值栈
- 这样可以处理重复最大值的情况
- 减少了最大值栈的空间占用

### 5. 扩展知识点

**其他实现方式**：

1. **单栈 + 最大值变量**：但无法处理重复最大值
2. **每个节点存储当前最大值**：空间换时间的另一种方式
3. **使用链表实现**：动态大小，但需要额外的指针空间

**实际应用场景**：

- 股票价格监控系统（实时获取最高价）
- 游戏排行榜系统
- 数据流处理中的滑动窗口最大值问题

**面试常见追问**：

- 如何支持getMin操作？（使用类似的辅助栈）
- 如何在多线程环境下保证线程安全？（加锁或使用并发数据结构）
- 能否进一步优化空间复杂度？（可以考虑惰性删除等策略）

这个实现既满足了时间复杂度要求，又考虑了实际工程中的边界条件处理，是一个比较完整的解决方案。


import java.util.EmptyStackException;

/**
 * 支持常数时间复杂度的push、pop、max操作的栈实现
 * 使用两个数组分别存储数据和最大值
 */
public class MaxStack {
    private int[] dataStack;      // 存储实际数据的栈
    private int[] maxStack;       // 存储最大值的辅助栈
    private int dataTop;          // 数据栈顶指针
    private int maxTop;           // 最大值栈顶指针
    private int capacity;         // 栈容量
    
    /**
     * 构造函数，初始化栈
     * @param capacity 栈的初始容量
     */
    public MaxStack(int capacity) {
        this.capacity = capacity;
        this.dataStack = new int[capacity];
        this.maxStack = new int[capacity];
        this.dataTop = -1;
        this.maxTop = -1;
    }
    
    /**
     * 入栈操作 - O(1)时间复杂度
     * @param value 要入栈的值
     */
    public void push(int value) {
        if (dataTop == capacity - 1) {
            throw new StackOverflowError("Stack is full");
        }
        
        // 将值压入数据栈
        dataStack[++dataTop] = value;
        
        // 更新最大值栈
        if (maxTop == -1 || value >= maxStack[maxTop]) {
            maxStack[++maxTop] = value;
        }
    }
    
    /**
     * 出栈操作 - O(1)时间复杂度
     * @return 栈顶元素
     */
    public int pop() {
        if (dataTop == -1) {
            throw new EmptyStackException();
        }
        
        int value = dataStack[dataTop--];
        
        // 如果弹出的值是当前最大值，则最大值栈也要弹出
        if (maxTop != -1 && value == maxStack[maxTop]) {
            maxTop--;
        }
        
        return value;
    }
    
    /**
     * 获取栈中最大值 - O(1)时间复杂度
     * @return 栈中的最大值
     */
    public int max() {
        if (maxTop == -1) {
            throw new EmptyStackException();
        }
        return maxStack[maxTop];
    }
    
    /**
     * 查看栈顶元素但不弹出 - O(1)时间复杂度
     * @return 栈顶元素
     */
    public int peek() {
        if (dataTop == -1) {
            throw new EmptyStackException();
        }
        return dataStack[dataTop];
    }
    
    /**
     * 检查栈是否为空
     * @return 栈是否为空
     */
    public boolean isEmpty() {
        return dataTop == -1;
    }
    
    /**
     * 获取栈的大小
     * @return 栈中元素个数
     */
    public int size() {
        return dataTop + 1;
    }
    
    /**
     * 打印栈的状态（用于调试）
     */
    public void printStack() {
        System.out.print("Data Stack: [");
        for (int i = 0; i <= dataTop; i++) {
            System.out.print(dataStack[i]);
            if (i < dataTop) System.out.print(", ");
        }
        System.out.println("]");
        
        System.out.print("Max Stack:  [");
        for (int i = 0; i <= maxTop; i++) {
            System.out.print(maxStack[i]);
            if (i < maxTop) System.out.print(", ");
        }
        System.out.println("]");
        System.out.println("Current Max: " + (maxTop == -1 ? "None" : maxStack[maxTop]));
        System.out.println("---");
    }
}

/**
 * 测试类
 */
class MaxStackTest {
    public static void main(String[] args) {
        MaxStack stack = new MaxStack(10);
        
        System.out.println("=== 测试MaxStack ===");
        
        // 测试push和max操作
        System.out.println("1. 依次推入: 3, 1, 4, 1, 5");
        stack.push(3);
        stack.printStack();
        
        stack.push(1);
        stack.printStack();
        
        stack.push(4);
        stack.printStack();
        
        stack.push(1);
        stack.printStack();
        
        stack.push(5);
        stack.printStack();
        
        // 测试pop操作
        System.out.println("2. 测试pop操作:");
        System.out.println("Pop: " + stack.pop()); // 应该返回5
        stack.printStack();
        
        System.out.println("Pop: " + stack.pop()); // 应该返回1
        stack.printStack();
        
        System.out.println("Pop: " + stack.pop()); // 应该返回4
        stack.printStack();
        
        // 测试重复最大值
        System.out.println("3. 测试重复最大值:");
        stack.push(3);
        stack.push(3);
        stack.push(2);
        stack.printStack();
        
        System.out.println("Pop: " + stack.pop()); // 应该返回2
        stack.printStack();
        
        System.out.println("Current max: " + stack.max()); // 应该是3
        
        // 性能测试
        System.out.println("\n4. 性能验证:");
        long startTime = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            if (!stack.isEmpty()) {
                stack.max(); // O(1)操作
            }
        }
        long endTime = System.nanoTime();
        System.out.println("100万次max操作耗时: " + (endTime - startTime) / 1000000.0 + " ms");
    }
}