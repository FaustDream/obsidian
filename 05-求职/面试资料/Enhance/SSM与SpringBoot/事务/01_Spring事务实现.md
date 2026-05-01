# Spring事务实现

## Spring事务实现机制详解

Spring事务是企业级Java开发中的核心机制，我来从底层原理到实际应用全面解析。

### 核心实现原理

**1. 事务管理器（PlatformTransactionManager）**

```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition);
    void commit(TransactionStatus status);
    void rollback(TransactionStatus status);
}
```

Spring提供了多种事务管理器实现：

- `DataSourceTransactionManager`：JDBC事务管理
- `JpaTransactionManager`：JPA事务管理
- `JtaTransactionManager`：分布式事务管理

**2. AOP代理机制**

Spring事务基于AOP实现，通过代理模式在方法调用前后织入事务逻辑：

```java
@Service
public class UserService {
    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
        // 业务逻辑
        accountDao.debit(fromId, amount);
        accountDao.credit(toId, amount);
    }
}
```

当调用`transferMoney`方法时，Spring会：

1. 创建代理对象
2. 在方法执行前开启事务
3. 执行业务方法
4. 根据执行结果提交或回滚事务

### 事务传播机制

Spring定义了7种事务传播行为：

```java
public enum Propagation {
    REQUIRED,        // 默认：存在事务就加入，不存在就创建
    SUPPORTS,        // 支持当前事务，没有就以非事务方式执行
    MANDATORY,       // 必须在事务中执行，否则抛异常
    REQUIRES_NEW,    // 总是创建新事务，挂起当前事务
    NOT_SUPPORTED,   // 以非事务方式执行，挂起当前事务
    NEVER,          // 以非事务方式执行，存在事务就抛异常
    NESTED          // 嵌套事务
}
```

**实际应用示例：**

```java
@Transactional
public void parentMethod() {
    // 外层事务
    childMethod1();  // REQUIRED - 加入当前事务
    childMethod2();  // REQUIRES_NEW - 创建新事务
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void childMethod2() {
    // 独立事务，即使外层回滚也不受影响
}
```

### 事务隔离级别

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void businessMethod() {
    // 读已提交级别，避免脏读
}
```

四种隔离级别对应的问题：

- `READ_UNCOMMITTED`：脏读、不可重复读、幻读
- `READ_COMMITTED`：不可重复读、幻读
- `REPEATABLE_READ`：幻读
- `SERIALIZABLE`：性能最差但最安全

### 声明式vs编程式事务

**声明式事务（推荐）：**

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    rollbackFor = Exception.class
)
public void declarativeTransaction() {
    // 业务逻辑
}
```

**编程式事务：**

```java
@Autowired
private TransactionTemplate transactionTemplate;

public void programmaticTransaction() {
    transactionTemplate.execute(status -> {
        try {
            // 业务逻辑
            return result;
        } catch (Exception e) {
            status.setRollbackOnly();
            throw e;
        }
    });
}
```

### 事务失效的常见场景

**1. 方法不是public**

```java
@Transactional
private void privateMethod() {
    // 事务不会生效
}
```

**2. 类内部方法调用**

```java
@Service
public class UserService {
    public void methodA() {
        methodB(); // 直接调用，不走代理，事务失效
    }
    
    @Transactional
    public void methodB() {
        // 事务不会生效
    }
}
```

**解决方案：**

```java
@Autowired
private UserService userService; // 注入自己

public void methodA() {
    userService.methodB(); // 通过代理调用
}
```

**3. 异常被catch但未重新抛出**

```java
@Transactional
public void methodWithException() {
    try {
        // 可能抛异常的代码
        riskyOperation();
    } catch (Exception e) {
        // 异常被吃掉，事务不会回滚
        log.error("Error occurred", e);
    }
}
```

### 分布式事务解决方案

**1. 两阶段提交（2PC）**

```java
@Transactional
@JtaTransactional
public void distributedTransaction() {
    // 跨多个数据源的操作
    dataSource1.update();
    dataSource2.update();
}
```

**2. TCC模式**

```java
@TccTransaction
public void tccTransaction() {
    // Try-Confirm-Cancel模式
}
```

**3. 消息最终一致性**

```java
@Transactional
public void reliableMessage() {
    // 本地事务
    orderService.createOrder();
    // 发送可靠消息
    messageProducer.sendReliableMessage();
}
```

### 性能优化建议

**1. 事务范围最小化**

```java
// 不好的做法
@Transactional
public void processLargeDataset() {
    for (int i = 0; i < 10000; i++) {
        heavyComputation(); // 耗时操作
        database.save();
    }
}

// 好的做法
public void processLargeDataset() {
    for (int i = 0; i < 10000; i++) {
        heavyComputation();
        saveWithTransaction(); // 只在需要时开启事务
    }
}

@Transactional
private void saveWithTransaction() {
    database.save();
}
```

**2. 只读事务优化**

```java
@Transactional(readOnly = true)
public List<User> queryUsers() {
    // 只读事务，数据库可以优化
    return userDao.findAll();
}
```

**3. 合理设置超时时间**

```java
@Transactional(timeout = 10) // 10秒超时
public void timeoutTransaction() {
    // 避免长时间锁定资源
}
```

Spring事务机制通过AOP和代理模式，提供了强大而灵活的事务管理能力。理解其实现原理和最佳实践，能够帮助我们构建更可靠的企业级应用。