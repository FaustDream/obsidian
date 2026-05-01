# Spring事务失效场景

Spring事务失效是Java面试中的高频考点。我来详细解析几种常见的失效场景及其原理：

## 1. 访问权限问题

**失效场景：** 事务方法不是public

```java
@Service
public class UserService {
    @Transactional
    private void updateUser() { // private方法，事务失效
        // 业务逻辑
    }
}
```

**原理：** Spring AOP默认使用JDK动态代理或CGLIB，只能代理public方法。

## 2. 方法被final修饰

```java
@Service
public class UserService {
    @Transactional
    public final void updateUser() { // final方法，CGLIB无法重写
        // 业务逻辑
    }
}
```

## 3. 同一个类中方法调用

**最常见的失效场景：**

```java
@Service
public class UserService {
    public void methodA() {
        this.methodB(); // 直接调用，绕过代理，事务失效
    }
    
    @Transactional
    public void methodB() {
        // 事务操作
    }
}
```

**解决方案：**

```java
// 方案1：注入自己
@Autowired
private UserService userService;

public void methodA() {
    userService.methodB(); // 通过代理调用
}

// 方案2：使用AopContext
public void methodA() {
    ((UserService) AopContext.currentProxy()).methodB();
}
```

## 4. 异常处理不当

### 4.1 异常被捕获未抛出

```java
@Transactional
public void updateUser() {
    try {
        // 可能抛异常的操作
        userMapper.update();
    } catch (Exception e) {
        log.error("更新失败", e); // 只记录日志，未重新抛出
        // 事务不会回滚
    }
}
```

### 4.2 抛出检查异常

```java
@Transactional
public void updateUser() throws Exception {
    throw new Exception("业务异常"); // 检查异常，默认不回滚
}

// 正确做法
@Transactional(rollbackFor = Exception.class)
public void updateUser() throws Exception {
    throw new Exception("业务异常");
}
```

## 5. 数据库引擎不支持事务

```java
// MySQL的MyISAM引擎不支持事务
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(50)
) ENGINE=MyISAM; // 应该使用InnoDB
```

## 6. 传播行为配置错误

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void updateUser() {
    // 以非事务方式执行，即使外层有事务也会挂起
}
```

## 7. 多线程场景

```java
@Transactional
public void updateUser() {
    CompletableFuture.runAsync(() -> {
        // 新线程中无法获取到事务上下文
        userMapper.update();
    });
}
```

## 8. Bean未被Spring管理

```java
// 手动new的对象，不是Spring代理对象
UserService userService = new UserService();
userService.updateUser(); // 事务失效
```

## 扩展知识点

### 事务的底层原理

1. **AOP代理**：Spring通过AOP创建代理对象
2. **ThreadLocal**：事务信息存储在ThreadLocal中
3. **数据库连接**：同一事务中共享数据库连接

### 事务传播机制

- **REQUIRED**：默认，如果有事务就加入，没有就新建
- **REQUIRES_NEW**：总是新建事务
- **NESTED**：嵌套事务，基于Savepoint实现

### 调试技巧

```java
// 判断当前方法是否在事务中
if (TransactionSynchronizationManager.isActualTransactionActive()) {
    log.info("当前在事务中");
}
```

这些场景在实际开发中都可能遇到，理解其原理有助于避免事务失效的问题，同时也是面试官考查Spring框架掌握程度的重要指标。