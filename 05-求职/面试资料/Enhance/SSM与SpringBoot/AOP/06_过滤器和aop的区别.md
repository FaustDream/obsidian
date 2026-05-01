# 过滤器和aop的区别

Filter（过滤器）和AOP（面向切面编程）都是Java Web开发中用于横切关注点处理的技术，但它们在实现机制、作用范围和使用场景上有显著区别。

## 主要区别

### 1. 技术层面和作用范围

**Filter（过滤器）**：

- 属于Servlet规范，是Web容器级别的技术
- 作用于HTTP请求/响应的整个生命周期
- 在请求到达Servlet之前和响应返回客户端之后执行
- 只能拦截HTTP请求，无法拦截Spring Bean的方法调用

**AOP（面向切面编程）**：

- 属于Spring框架的核心特性
- 作用于Spring容器管理的Bean的方法级别
- 可以在方法执行前、后、异常时等多个切入点执行
- 能够拦截任意Spring管理的Bean方法

### 2. 实现机制

**Filter实现**：

```java
@Component
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        // 请求前处理
        System.out.println("请求开始：" + ((HttpServletRequest)request).getRequestURI());
        
        // 执行后续过滤器或目标资源
        chain.doFilter(request, response);
        
        // 响应后处理
        System.out.println("请求结束");
    }
}
```

**AOP实现**：

```java
@Aspect
@Component
public class LoggingAspect {
    @Around("@annotation(Loggable)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();
        
        long executionTime = System.currentTimeMillis() - start;
        System.out.println("方法执行时间：" + executionTime + "ms");
        
        return result;
    }
}
```

### 3. 配置和管理

**Filter配置**：

```java
// 注解方式
@WebFilter(urlPatterns = "/*", filterName = "loggingFilter")

// 或者在Spring Boot中通过FilterRegistrationBean
@Bean
public FilterRegistrationBean<LoggingFilter> loggingFilter() {
    FilterRegistrationBean<LoggingFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new LoggingFilter());
    registrationBean.addUrlPatterns("/*");
    registrationBean.setOrder(1);
    return registrationBean;
}
```

**AOP配置**：

```java
// 启用AOP
@EnableAspectJAutoProxy
@Configuration
public class AopConfig {
    // Spring Boot中默认自动配置
}
```

## 使用场景对比

### Filter适合的场景：

- **字符编码设置**：统一设置请求/响应的字符编码
- **跨域处理**：添加CORS相关的响应头
- **身份认证**：检查用户登录状态
- **请求日志**：记录所有HTTP请求的基本信息
- **请求参数预处理**：对所有请求参数进行统一处理

### AOP适合的场景：

- **方法执行时间监控**：精确测量业务方法执行时间
- **缓存管理**：方法级别的缓存控制
- **事务管理**：声明式事务处理
- **权限校验**：基于方法或类的细粒度权限控制
- **异常处理**：统一的异常捕获和处理

## 执行顺序

在一个完整的请求处理流程中：

```
客户端请求 
   ↓
Filter1 (前置处理)
   ↓  
Filter2 (前置处理)
   ↓
DispatcherServlet
   ↓
AOP前置通知 (@Before)
   ↓
Controller方法执行
   ↓
AOP后置通知 (@After)
   ↓
Filter2 (后置处理)
   ↓
Filter1 (后置处理)
   ↓
响应返回客户端
```

## 性能考虑

- **Filter**：对所有匹配的HTTP请求都会执行，相对开销较大
- **AOP**：只对目标方法执行，可以精确控制切入点，性能开销更小

## 实际应用建议

1. **Web层面的横切关注点**使用Filter：如安全检查、编码设置、CORS处理
2. **业务层面的横切关注点**使用AOP：如日志记录、性能监控、事务管理
3. **两者结合使用**：Filter处理Web容器级别的需求，AOP处理业务逻辑级别的需求

这样的设计既发挥了各自的优势，又避免了功能重复，使系统架构更加清晰和高效。