# 你的项目中的隔离级别如何设置，在service层加事务注解会有什么问题

在Java项目中，事务隔离级别的设置和Service层事务注解的使用是非常重要的话题。让我详细为你解答：

## 隔离级别设置方式

### 1. 全局配置

```yaml
# application.yml
spring:
  datasource:
    hikari:
      transaction-isolation: TRANSACTION_READ_COMMITTED
```

### 2. 注解级别设置

```java
@Service
@Transactional(isolation = Isolation.READ_COMMITTED)
public class UserService {
    
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void updateUser(User user) {
        // 方法级别可以覆盖类级别设置
    }
}
```

## Service层加事务注解的潜在问题

### 1. **事务传播性问题**

```java
@Service
public class UserService {
    
    @Transactional
    public void updateUserInfo() {
        updateUser();     // 内部调用不会开启新事务
        updateProfile();  // 可能导致部分回滚问题
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    private void updateUser() {
        // 私有方法上的@Transactional不生效！
    }
}
```

**解决方案**：

- 将内部调用的方法提取到另一个Service
- 使用`self-injection`模式
- 使用编程式事务

### 2. **性能问题**

```java
@Service
@Transactional  // 类级别注解导致所有方法都有事务开销
public class UserService {
    
    public List<User> queryUsers() {
        // 只读操作不需要事务，但仍然有事务开销
        return userDao.selectAll();
    }
}
```

**优化方案**：

```java
@Service
public class UserService {
    
    @Transactional(readOnly = true)
    public List<User> queryUsers() {
        return userDao.selectAll();
    }
    
    @Transactional
    public void updateUser(User user) {
        userDao.update(user);
    }
}
```

### 3. **异常处理问题**

```java
@Service
public class UserService {
    
    @Transactional
    public void processUser(User user) {
        try {
            userDao.save(user);
            // checked exception不会回滚事务
            throw new Exception("业务异常");
        } catch (Exception e) {
            // 事务不会回滚！
            log.error("处理失败", e);
        }
    }
}
```

**正确做法**：

```java
@Transactional(rollbackFor = Exception.class)
public void processUser(User user) throws Exception {
    userDao.save(user);
    // 让异常向上抛出，或者手动标记回滚
    if (someCondition) {
        TransactionAspectSupport.currentTransactionStatus()
            .setRollbackOnly();
    }
}
```

### 4. **长事务问题**

```java
@Service
public class ReportService {
    
    @Transactional
    public void generateReport() {
        // 大量数据处理
        for (int i = 0; i < 100000; i++) {
            processData(i);  // 长时间占用数据库连接
        }
        // 可能导致死锁或连接池耗尽
    }
}
```

## 最佳实践建议

### 1. **合理选择隔离级别**

```java
// 根据业务场景选择
@Transactional(isolation = Isolation.READ_COMMITTED)  // 防止脏读
public void normalBusiness() {}

@Transactional(isolation = Isolation.REPEATABLE_READ) // 防止不可重复读
public void criticalBusiness() {}
```

### 2. **精细化事务控制**

```java
@Service
public class OrderService {
    
    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        return orderDao.findById(id);
    }
    
    @Transactional(
        isolation = Isolation.READ_COMMITTED,
        propagation = Propagation.REQUIRED,
        timeout = 30,
        rollbackFor = Exception.class
    )
    public void createOrder(Order order) {
        orderDao.save(order);
        inventoryService.reduceStock(order.getItems());
    }
}
```

### 3. **避免大事务**

```java
@Service
public class BatchService {
    
    @Autowired
    private BatchProcessor batchProcessor; // 注入另一个Service处理事务
    
    public void batchProcess(List<Data> dataList) {
        Lists.partition(dataList, 100).forEach(batch -> {
            try {
                batchProcessor.processBatch(batch);
            } catch (Exception e) {
                // 记录异常，继续处理下一批次
                log.error("批次处理失败", e);
            }
        });
    }
}

@Service
public class BatchProcessor {
    
    @Transactional(rollbackFor = Exception.class)
    public void processBatch(List<Data> batch) {
        batch.forEach(data -> processData(data));
    }
}
```

或使用手动事务

```java
@Service
public class BatchService {
    
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public void batchProcess(List<Data> dataList) {
        Lists.partition(dataList, 100).forEach(batch -> {
            transactionTemplate.executeWithoutResult(status -> {
                try {
                    batch.forEach(data -> processData(data));
                } catch (Exception e) {
                    status.setRollbackOnly();
                    throw e;
                }
            });
        });
    }
}
```

这些问题在实际项目中非常常见，正确理解和使用事务注解能够显著提升系统的稳定性和性能。