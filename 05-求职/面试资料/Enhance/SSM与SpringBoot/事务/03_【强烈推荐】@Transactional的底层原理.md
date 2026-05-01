# @Transactional的底层原理？

## @Transactional 底层原理详解

### 核心实现机制

**@Transactional 基于 Spring AOP（面向切面编程）实现**，主要通过以下步骤工作：

#### 1. 代理对象创建

Spring 容器启动时，会扫描所有标注了 @Transactional 的类和方法，然后为这些类创建代理对象：

- **JDK 动态代理**：当目标类实现了接口时使用
- **CGLIB 代理**：当目标类没有实现接口时使用

#### 2. 事务拦截器工作流程

当调用被 @Transactional 标注的方法时，实际调用的是代理对象，执行顺序如下：

```java
// 伪代码展示拦截器逻辑
public Object invoke() {
    TransactionInfo txInfo = null;
    try {
        // 1. 开启事务
        txInfo = createTransactionIfNecessary();
        
        // 2. 执行目标方法
        Object result = method.invoke(target, args);
        
        // 3. 提交事务
        commitTransactionAfterReturning(txInfo);
        return result;
        
    } catch (Exception ex) {
        // 4. 回滚事务
        completeTransactionAfterThrowing(txInfo, ex);
        throw ex;
    } finally {
        // 5. 清理事务信息
        cleanupTransactionInfo(txInfo);
    }
}
```

### 关键组件解析

#### 1. PlatformTransactionManager

事务管理器接口，负责事务的创建、提交、回滚：

```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition);
    void commit(TransactionStatus status);
    void rollback(TransactionStatus status);
}
```

#### 2. TransactionDefinition

定义事务属性：

- **传播行为**（Propagation）：REQUIRED、REQUIRES_NEW 等
- **隔离级别**（Isolation）：READ_COMMITTED、REPEATABLE_READ 等
- **超时时间**（Timeout）
- **只读标识**（ReadOnly）

#### 3. TransactionStatus

表示事务运行状态，包含事务是否为新事务、是否已完成等信息

### 事务传播机制

**REQUIRED**（默认）：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    methodB(); // 加入当前事务
}
```

**REQUIRES_NEW**：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // 创建新事务，挂起当前事务
}
```

### 数据库层面实现

Spring 事务最终会转换为数据库层面的操作：

1. **开启事务**：`BEGIN` 或设置 `autocommit=false`
2. **提交事务**：`COMMIT`
3. **回滚事务**：`ROLLBACK`

### 重要注意事项

#### 1. 失效场景

```java
@Service
public class UserService {
    // ❌ 同类内部调用，事务失效
    public void methodA() {
        this.methodB(); // 直接调用，绕过代理
    }
    
    @Transactional
    public void methodB() {
        // 事务不会生效
    }
}
```

#### 2. 异常处理

```java
@Transactional(rollbackFor = Exception.class)
public void updateUser() {
    // 默认只对 RuntimeException 和 Error 回滚
    // 需要显式指定 rollbackFor 来处理检查异常
}
```

### 扩展知识点

#### 1. 事务同步机制

Spring 提供 `TransactionSynchronization` 接口，允许在事务的不同阶段执行自定义逻辑：

```java
TransactionSynchronizationManager.registerSynchronization(
    new TransactionSynchronization() {
        @Override
        public void afterCommit() {
            // 事务提交后执行
        }
    }
);
```

#### 2. 分布式事务

在微服务架构中，可能需要：

- **两阶段提交（2PC）**
- **Saga 模式**
- **TCC（Try-Confirm-Cancel）模式**

#### 3. 性能优化

- 合理设置事务边界，避免长事务
- 使用只读事务优化查询性能
- 考虑异步处理减少事务持有时间

这套机制确保了数据一致性，同时提供了灵活的事务管理能力，是 Spring 框架的核心特性之一。