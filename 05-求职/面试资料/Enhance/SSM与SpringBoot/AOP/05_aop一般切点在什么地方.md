# aop一般切点在什么地方

AOP（面向切面编程）的切点通常设置在以下几个关键位置：

## 常见切点位置

**1. Service层方法**

- 业务逻辑方法的入口和出口
- 适合添加事务管理、日志记录、性能监控

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}
```

**2. Controller层方法**

- Web请求的处理入口
- 适合做权限验证、参数校验、请求日志

```java
@Pointcut("@annotation(org.springframework.web.bind.annotation.RequestMapping)")
public void controllerMethods() {}
```

**3. Repository/DAO层方法**

- 数据访问层
- 适合做数据库操作日志、异常处理

```java
@Pointcut("execution(* com.example.repository.*.*(..))")
public void repositoryLayer() {}
```

## 具体应用场景

**事务管理**

```java
@Around("@annotation(Transactional)")
public Object handleTransaction(ProceedingJoinPoint joinPoint) {
    // 事务开启、提交、回滚逻辑
}
```

**日志记录**

```java
@Around("execution(* com.example.service.*.*(..))")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    logger.info("方法执行时间: {}ms", end - start);
    return result;
}
```

**权限控制**

```java
@Before("@annotation(RequiresPermission)")
public void checkPermission(JoinPoint joinPoint) {
    // 权限验证逻辑
}
```

## 选择切点的原则

1. **边界清晰**：选择架构层次分明的地方
2. **职责单一**：避免在业务逻辑复杂的方法内部切入
3. **影响可控**：确保切面逻辑不会破坏原有业务
4. **性能考虑**：避免在高频调用的细粒度方法上设置切点

最佳实践是在各个架构层的边界处设置切点，这样既能实现横切关注点的统一处理，又不会过度侵入业务逻辑。