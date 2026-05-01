## AOP原理及各种Advice

### AOP核心原理

**AOP（面向切面编程）** 是一种编程范式，通过 **横向关注点分离** 来增强代码的模块化。其核心思想是将业务逻辑和系统级关注点（如日志、事务、安全等）分离。

#### 实现原理

AOP主要通过**动态代理**实现：

**1. JDK动态代理**

- 基于接口实现
- 使用`java.lang.reflect.Proxy`
- 目标对象必须实现接口

```java
// 示例：JDK动态代理
public class JdkProxyExample implements InvocationHandler {
    private Object target;
    
    public Object invoke(Object proxy, Method method, Object[] args) {
        // 前置增强
        System.out.println("Before method: " + method.getName());
        Object result = method.invoke(target, args);
        // 后置增强
        System.out.println("After method: " + method.getName());
        return result;
    }
}
```

**2. CGLIB动态代理**

- 基于继承实现
- 使用字节码技术生成子类
- 无需接口，==但不能代理final类/方法==

```java
// CGLIB代理示例
public class CglibMethodInterceptor implements MethodInterceptor {
    public Object intercept(Object obj, Method method, Object[] args, 
                          MethodProxy proxy) throws Throwable {
        System.out.println("Before: " + method.getName());
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After: " + method.getName());
        return result;
    }
}
```

### 各种Advice类型

#### 1. Before Advice（前置通知）

在目标方法执行**之前**执行

```java
@Aspect
@Component
public class BeforeAdviceExample {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void beforeAdvice(JoinPoint joinPoint) {
        System.out.println("执行前置通知：" + joinPoint.getSignature().getName());
        // 参数校验、权限检查等
    }
}
```

**应用场景**：参数校验、权限检查、日志记录

#### 2. After Returning Advice（后置返回通知）

在目标方法**正常返回**后执行

```java
@AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", 
                returning = "result")
public void afterReturningAdvice(JoinPoint joinPoint, Object result) {
    System.out.println("方法正常返回：" + result);
    // 结果处理、缓存更新等
}
```

**应用场景**：结果处理、缓存更新、成功日志

#### 3. After Throwing Advice（异常通知）

在目标方法**抛出异常**后执行

```java
@AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", 
               throwing = "ex")
public void afterThrowingAdvice(JoinPoint joinPoint, Exception ex) {
    System.out.println("方法异常：" + ex.getMessage());
    // 异常日志、邮件通知等
}
```

**应用场景**：异常日志记录、错误通知、资源清理

#### 4. After Finally Advice（最终通知）

无论目标方法是否正常执行都会执行

```java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice(JoinPoint joinPoint) {
    System.out.println("最终通知，无论成功失败都执行");
    // 资源释放、统计信息等
}
```

**应用场景**：资源释放、执行时间统计、清理工作

#### 5. Around Advice（环绕通知）

**最强大的通知类型**，可以控制目标方法的执行

```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    long startTime = System.currentTimeMillis();
    
    try {
        System.out.println("环绕通知-前置");
        
        // 调用目标方法
        Object result = joinPoint.proceed();
        
        System.out.println("环绕通知-后置返回");
        return result;
        
    } catch (Exception e) {
        System.out.println("环绕通知-异常处理");
        throw e;
    } finally {
        long endTime = System.currentTimeMillis();
        System.out.println("方法执行时间：" + (endTime - startTime) + "ms");
    }
}
```

**应用场景**：性能监控、事务管理、缓存控制

### 执行顺序

当多种Advice同时作用于一个方法时，执行顺序如下：

```
Around Before → Before → 目标方法 → Around After → After → AfterReturning/AfterThrowing
```

### 实际应用示例

#### 统一异常处理

```java
@Aspect
@Component
public class ExceptionHandlingAspect {
    
    @AfterThrowing(pointcut = "execution(* com.example.controller.*.*(..))", 
                   throwing = "ex")
    public void handleException(JoinPoint joinPoint, Exception ex) {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        
        // 记录异常日志
        log.error("异常发生在 {}.{}: {}", className, methodName, ex.getMessage());
        
        // 发送邮件通知
        emailService.sendExceptionAlert(className, methodName, ex);
    }
}
```

#### 性能监控

```java
@Aspect
@Component
public class PerformanceMonitorAspect {
    
    @Around("@annotation(com.example.annotation.Monitor)")
    public Object monitor(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        
        try {
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            String methodName = joinPoint.getSignature().getName();
            log.info("方法 {} 执行时间：{}ms", methodName, stopWatch.getTotalTimeMillis());
        }
    }
}
```

### 扩展知识点

#### 1. Pointcut表达式

- `execution()`：方法执行
- `within()`：类型匹配
- `@annotation()`：注解匹配
- `args()`：参数匹配

#### 2. AOP vs OOP

- **OOP**：纵向继承，解决代码重用
- **AOP**：横向切入，解决横切关注点

#### 3. Spring AOP限制

- 只能代理Spring容器中的Bean
- 内部方法调用不会触发AOP
- 只支持方法级别的切入点

#### 4. AspectJ vs Spring AOP

- **Spring AOP**：运行时代理，功能有限但易用
- **AspectJ**：编译时增强，功能强大但复杂

这样的设计让我们能够在不修改原有业务代码的情况下，优雅地添加横切关注点的处理逻辑，真正实现了关注点分离的目标。