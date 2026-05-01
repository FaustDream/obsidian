# AOP原理，jdk和cglib哪个更快？AOP用途

## AOP原理

AOP（面向切面编程）的核心原理是**代理模式**。它通过在目标对象外层创建代理对象，在方法调用前后插入横切逻辑（如日志、事务、安全检查等）。

**工作流程：**

1. 容器启动时，AOP框架扫描带有切面注解的类和方法
2. 为目标对象创建代理对象，代理对象包装了原始对象
3. 当调用目标方法时，实际调用的是代理对象的方法
4. 代理对象根据切点匹配规则，在适当时机执行通知（Advice）
5. 最终调用目标对象的真实方法

## JDK动态代理 vs CGLIB性能对比

**JDK动态代理更快**，主要原因：

### JDK动态代理优势：

- **原生支持**：JVM内置优化，字节码生成效率高
- **接口调用**：基于接口的方法调用，JVM优化程度更高
- **内存占用小**：生成的代理类相对简单
- **创建速度快**：Proxy.newProxyInstance()执行效率高

### CGLIB劣势：

- **字节码操作**：需要生成更复杂的子类字节码
- **ASM依赖**：底层依赖ASM库进行字节码操作，有额外开销
- **类加载开销**：生成的子类需要被类加载器加载
- **方法调用链更长**：需要通过继承关系调用父类方法

**性能测试结果**：在大多数场景下，JDK动态代理比CGLIB快10%-30%。

## AOP的主要用途

### 1. **事务管理**

```java
@Transactional
public void transferMoney(Account from, Account to, BigDecimal amount) {
    // 业务逻辑，AOP自动处理事务开启、提交、回滚
}
```

### 2. **日志记录**

```java
@Around("@annotation(Log)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    logger.info("方法{}执行耗时: {}ms", joinPoint.getSignature().getName(), end - start);
    return result;
}
```

### 3. **权限控制**

```java
@Before("@annotation(RequireRole)")
public void checkPermission(JoinPoint joinPoint) {
    // 检查用户权限
    if (!hasPermission(getCurrentUser(), requiredRole)) {
        throw new AccessDeniedException("权限不足");
    }
}
```

### 4. **缓存管理**

```java
@Around("@annotation(Cacheable)")
public Object cache(ProceedingJoinPoint joinPoint) {
    String key = generateKey(joinPoint.getArgs());
    Object cached = cacheManager.get(key);
    if (cached != null) return cached;
    
    Object result = joinPoint.proceed();
    cacheManager.put(key, result);
    return result;
}
```

### 5. **异常处理**

- 统一异常日志记录
- 异常转换和包装
- 失败重试机制

### 6. **性能监控**

- 方法执行时间统计
- 调用次数统计
- 系统性能指标收集

## 扩展知识

### Spring AOP选择策略：

- **有接口**：优先使用JDK动态代理
- **无接口**：使用CGLIB
- **可强制指定**：通过`proxy-target-class=true`强制使用CGLIB

### 注意事项：

1. **自调用问题**：同一个类内部方法调用不会触发AOP
2. **private方法**：无法被代理
3. **final类/方法**：CGLIB无法代理
4. **性能考虑**：避免在高频调用的方法上使用复杂切面逻辑

AOP作为Spring框架的核心特性之一，极大地提高了代码的模块化程度，让开发者能够专注于业务逻辑，而将横切关注点交给框架处理。