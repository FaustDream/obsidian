# 事务ACID和隔离级别

## 事务ACID特性

ACID是数据库事务必须具备的四个基本特性：

### A - 原子性（Atomicity）

事务是一个不可分割的工作单位，要么全部成功，要么全部失败回滚。

**举例**：银行转账操作

```java
@Transactional
public void transfer(String fromAccount, String toAccount, BigDecimal amount) {
    // 这两个操作要么都成功，要么都失败
    accountService.debit(fromAccount, amount);  // 扣款
    accountService.credit(toAccount, amount);   // 入账
}
```

### C - 一致性（Consistency）

事务执行前后，数据库必须保持一致性状态，不能违反任何完整性约束。

**举例**：上述转账中，转账前后总金额必须保持不变，账户余额不能为负数。

### I - 隔离性（Isolation）

并发执行的事务之间不能相互干扰，一个事务的中间状态对其他事务是不可见的。

### D - 持久性（Durability）

事务一旦提交，对数据库的改变就是永久性的，即使系统崩溃也不会丢失。

## 事务隔离级别

SQL标准定义了四种隔离级别，用来解决并发事务可能出现的问题：

### 1. 读未提交（READ UNCOMMITTED）

- **特点**：最低的隔离级别，可以读取到其他事务未提交的数据
- **问题**：会出现脏读、不可重复读、幻读
- **性能**：最高
- **使用场景**：几乎不使用，因为数据一致性无法保证

### 2. 读已提交（READ COMMITTED）

- **特点**：只能读取到已经提交的数据
- **解决**：脏读问题
- **问题**：仍会出现不可重复读、幻读
- **使用场景**：Oracle、SQL Server默认级别

```java
// 演示不可重复读问题
@Transactional(isolation = Isolation.READ_COMMITTED)
public void demo() {
    User user1 = userDao.findById(1);  // 读取到 name="张三"
    // 此时另一个事务修改了用户名并提交
    User user2 = userDao.findById(1);  // 读取到 name="李四"
    // 同一个事务中两次读取结果不一致
}
```

### 3. 可重复读（REPEATABLE READ）

- **特点**：保证在同一事务中多次读取同样记录的结果一致
- **解决**：脏读、不可重复读
- **问题**：仍会出现幻读
- **使用场景**：MySQL的InnoDB引擎默认级别

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void demo() {
    List<User> users1 = userDao.findAll();  // 查询到5条记录
    // 此时另一个事务插入了新记录并提交
    List<User> users2 = userDao.findAll();  // 可能查询到6条记录（幻读）
}
```

### 4. 序列化（SERIALIZABLE）

- **特点**：最高的隔离级别，完全串行化执行
- **解决**：脏读、不可重复读、幻读
- **问题**：性能最差，容易产生死锁
- **使用场景**：对数据一致性要求极高的场景

## 并发问题详解

### 脏读（Dirty Read）

读取到其他事务未提交的数据。

```java
// 事务A
BEGIN;
UPDATE account SET balance = 1000 WHERE id = 1;
// 此时事务B读取到balance=1000（脏读）
ROLLBACK;  // 事务A回滚

// 事务B
BEGIN;
SELECT balance FROM account WHERE id = 1;  // 读取到1000，但实际应该是原值
```

### 不可重复读（Non-Repeatable Read）

同一事务中两次读取同一行数据结果不同。

```java
// 事务A
BEGIN;
SELECT name FROM user WHERE id = 1;  // 读取到"张三"
// 此时事务B修改了name="李四"并提交
SELECT name FROM user WHERE id = 1;  // 读取到"李四"
COMMIT;
```

### 幻读（Phantom Read）

同一事务中两次查询返回的行数不同。

```java
// 事务A
BEGIN;
SELECT COUNT(*) FROM user WHERE age > 20;  // 返回5条
// 此时事务B插入了一条age=25的记录并提交
SELECT COUNT(*) FROM user WHERE age > 20;  // 返回6条
COMMIT;
```

## Spring中的使用

```java
@Service
public class OrderService {
    
    // 使用默认隔离级别
    @Transactional
    public void createOrder(Order order) {
        // 业务逻辑
    }
    
    // 指定隔离级别
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void updateOrder(Order order) {
        // 业务逻辑
    }
    
    // 只读事务，可以优化性能
    @Transactional(readOnly = true, isolation = Isolation.READ_COMMITTED)
    public List<Order> findOrders() {
        return orderDao.findAll();
    }
}
```

## 性能与选择建议

1. **读未提交**：性能最好，但数据一致性无法保证，实际中几乎不用
2. **读已提交**：适合大多数业务场景，Oracle默认级别
3. **可重复读**：MySQL默认级别，在大多数情况下是好的选择
4. **序列化**：只在对数据一致性要求极高时使用，注意死锁问题

**选择原则**：在满足业务需求的前提下，选择性能最好的隔离级别。大多数情况下，读已提交或可重复读就足够了。