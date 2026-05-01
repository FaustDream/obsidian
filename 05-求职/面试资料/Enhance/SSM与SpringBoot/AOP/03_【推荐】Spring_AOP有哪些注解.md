# Spring AOP有哪些注解？

Spring AOP 主要包含以下几类注解：

## 切面相关注解

**@Aspect** - 声明一个类为切面类

```java
@Aspect
@Component
public class LoggingAspect {
    // 切面逻辑
}
```

## 通知（Advice）注解

**@Before** - 前置通知，在目标方法执行前执行

```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice() {
    System.out.println("方法执行前");
}
```

**@After** - 后置通知，在目标方法执行后执行（无论是否抛异常）

```java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice() {
    System.out.println("方法执行后");
}
```

**@AfterReturning** - 返回通知，在目标方法正常返回后执行

```java
@AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
public void afterReturningAdvice(Object result) {
    System.out.println("方法返回值：" + result);
}
```

**@AfterThrowing** - 异常通知，在目标方法抛出异常后执行

```java
@AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", throwing = "ex")
public void afterThrowingAdvice(Exception ex) {
    System.out.println("方法抛出异常：" + ex.getMessage());
}
```

**@Around** - 环绕通知，可以在目标方法执行前后都执行逻辑

```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("方法执行前");
    Object result = joinPoint.proceed(); // 执行目标方法
    System.out.println("方法执行后");
    return result;
}
```

## 切点相关注解

**@Pointcut** - 定义可重用的切点表达式

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}

@Before("serviceLayer()")
public void beforeService() {
    // 使用定义好的切点
}
```

## 启用AOP的注解

**@EnableAspectJAutoProxy** - 在配置类上启用AspectJ自动代理
> Spring Boot 一般不需要显式标注此注解，仅在需要修改默认参数时显式标注。如：
> ```
> @EnableAspectJAutoProxy(
    proxyTargetClass = true,  // 强制使用CGLIB代理
    exposeProxy = true        // 暴露代理对象
   )
> ```

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    // 配置
}
```

## 扩展知识点

**执行顺序**：正常情况下为 @Around前 → @Before → 目标方法 → @AfterReturning → @After → @Around后

**代理机制**：Spring AOP默认使用JDK动态代理（接口）或CGLIB代理（类）

**切点表达式**：支持execution、within、this、target、args等多种表达式类型

**性能考虑**：AOP会增加一定的性能开销，应合理使用避免过度切面化

这些注解构成了Spring AOP的核心功能，通过声明式的方式实现了横切关注点的分离。