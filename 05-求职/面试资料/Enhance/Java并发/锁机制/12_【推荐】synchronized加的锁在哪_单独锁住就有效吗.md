# synchronized加的锁在哪，单独锁住就有效吗

`synchronized` 关键字的锁机制和锁对象的选择是 Java 并发编程中的核心概念，我来详细解释一下：

## synchronized 锁的位置

### 1. 修饰实例方法

```java
public synchronized void method() {
    // 锁对象是当前实例对象 (this)
}
```

### 2. 修饰静态方法

```java
public static synchronized void staticMethod() {
    // 锁对象是当前类的 Class 对象
}
```

### 3. 修饰代码块

```java
public void method() {
    synchronized(this) {  // 锁对象是 this
        // 临界区代码
    }
    
    synchronized(SomeClass.class) {  // 锁对象是 Class 对象
        // 临界区代码
    }
    
    synchronized(lockObject) {  // 锁对象是指定的对象
        // 临界区代码
    }
}
```

## 锁对象的存储位置

synchronized 的锁信息实际存储在 **对象头（Object Header）** 中：

```java
// 对象在内存中的结构
|--------------------------------------------------------------|
| Object Header (64/128 bits)                                 |
|--------------------------------------------------------------|
| Mark Word (32/64 bits) | Klass Word (32/64 bits)           |
|--------------------------------------------------------------|
| Instance Data                                                |
|--------------------------------------------------------------|
| Padding                                                      |
|--------------------------------------------------------------|
```

Mark Word 中包含了锁状态信息：

- **无锁状态**：存储对象的 hashCode、分代年龄等
- **轻量级锁**：存储锁记录的指针
- **重量级锁**：存储 monitor 对象的指针
- **偏向锁**：存储偏向线程 ID

## 单独锁住是否有效？

**答案是：不一定有效**。synchronized 的有效性取决于多个线程是否使用**同一个锁对象**。

### 有效的情况：

```java
public class Counter {
    private int count = 0;
    private Object lock = new Object();
    
    public void increment() {
        synchronized(lock) {  // 所有线程都使用同一个 lock 对象
            count++;
        }
    }
    
    public int getCount() {
        synchronized(lock) {  // 使用相同的锁对象
            return count;
        }
    }
}
```

### 无效的情况：

```java
public class BadCounter {
    private int count = 0;
    
    public void increment() {
        Object lock = new Object();  // 每次都创建新的锁对象
        synchronized(lock) {         // 实际上没有同步效果
            count++;
        }
    }
}
```

## 锁升级过程

JVM 对 synchronized 进行了优化，锁会根据竞争情况进行升级：

1. **偏向锁**：当只有一个线程访问时，偏向该线程
2. **轻量级锁**：当有少量线程竞争时，使用 CAS 操作
3. **重量级锁**：当竞争激烈时，使用操作系统的互斥量

## 实际应用示例

```java
public class BankAccount {
    private double balance;
    private final Object balanceLock = new Object();
    
    public void withdraw(double amount) {
        synchronized(balanceLock) {
            if (balance >= amount) {
                balance -= amount;
            }
        }
    }
    
    public void deposit(double amount) {
        synchronized(balanceLock) {  // 使用相同的锁对象
            balance += amount;
        }
    }
    
    public double getBalance() {
        synchronized(balanceLock) {  // 读操作也需要同步
            return balance;
        }
    }
}
```

## 关键要点

1. **锁对象必须相同**：所有需要同步的代码块必须使用同一个锁对象
2. **锁的粒度**：锁的范围应该尽可能小，避免不必要的性能损失
3. **避免死锁**：获取多个锁时要注意顺序，避免循环等待
4. **读写都要同步**：对共享变量的读写操作都应该在同步块中进行

synchronized 的本质是通过对象监视器（monitor）来实现线程同步，只有正确使用相同的锁对象才能保证线程安全。