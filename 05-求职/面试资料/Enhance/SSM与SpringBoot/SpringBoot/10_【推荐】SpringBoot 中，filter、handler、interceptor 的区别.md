在SpringBoot中，filter、handler、interceptor这三者都是处理请求的组件，但它们在架构层次、执行时机和功能范围上有明显区别。让我详细解释一下：

## 1. Filter（过滤器）

**定义与特点：**

- 属于Servlet规范，不是Spring特有的
- 在Servlet容器层面工作，位于SpringMVC之前
- 可以过滤所有请求，包括静态资源

**执行时机：**

- 请求到达DispatcherServlet之前执行
- 响应返回给客户端之前执行

**典型应用场景：**

```java
@Component
public class RequestLoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        // 请求处理前
        System.out.println("Filter: 请求开始");
        chain.doFilter(request, response);
        // 请求处理后
        System.out.println("Filter: 请求结束");
    }
}
```

**使用场景：**

- 字符编码设置
- 跨域处理
- 安全认证
- 请求日志记录

## 2. Interceptor（拦截器）

**定义与特点：**

- Spring MVC框架特有的组件
- 在DispatcherServlet内部工作
- 只能拦截Controller请求，无法拦截静态资源（除非特殊配置）

**执行时机：**

- preHandle：Controller方法执行前
- postHandle：Controller方法执行后，视图渲染前
- afterCompletion：整个请求完成后

**典型应用场景：**

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        // Controller执行前
        System.out.println("Interceptor: preHandle");
        return true; // 返回false会中断请求
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                          HttpServletResponse response, 
                          Object handler, 
                          ModelAndView modelAndView) throws Exception {
        // Controller执行后，视图渲染前
        System.out.println("Interceptor: postHandle");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler, Exception ex) throws Exception {
        // 请求完成后
        System.out.println("Interceptor: afterCompletion");
    }
}
```

**使用场景：**

- 用户权限验证
- 操作日志记录
- 性能监控
- 数据预处理

## 3. Handler（处理器）

**定义与特点：**

- 通常指Controller中的方法或其他处理请求的组件
- 是实际处理业务逻辑的地方
- 可以是@Controller、@RestController中的方法

**典型应用场景：**

```java
@RestController
public class UserController {
    @GetMapping("/user/{id}")
    public User getUser(@PathVariable Long id) {
        // Handler：实际的业务处理逻辑
        System.out.println("Handler: 处理获取用户请求");
        return userService.findById(id);
    }
}
```

## 执行顺序

完整的请求处理流程：

```
客户端请求 
    ↓
Filter（请求前处理）
    ↓
DispatcherServlet
    ↓
Interceptor.preHandle()
    ↓
Handler（Controller方法）
    ↓
Interceptor.postHandle()
    ↓
视图渲染
    ↓
Interceptor.afterCompletion()
    ↓
Filter（响应后处理）
    ↓
客户端接收响应
```

## 主要区别对比

|特性|Filter|Interceptor|Handler|
|---|---|---|---|
|**规范**|Servlet规范|Spring MVC|Spring MVC|
|**执行层次**|Servlet容器层|DispatcherServlet内|业务处理层|
|**拦截范围**|所有请求|Controller请求|具体业务逻辑|
|**Spring依赖**|无需Spring|需要Spring|需要Spring|
|**执行次数**|请求/响应各一次|可多个执行点|一次|

## 扩展知识点

**1. 多个Filter的执行顺序：**

```java
@Bean
@Order(1)
public FilterRegistrationBean<FirstFilter> firstFilter() {
    // Order值越小，优先级越高
}
```

**2. Interceptor配置：**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/login");
    }
}
```

**3. 异常处理：**

- Filter中的异常需要在Filter内处理
- Interceptor中的异常可以被Spring的异常处理机制捕获
- Handler中的异常可以通过@ExceptionHandler处理

这三者配合使用可以构建一个完整的请求处理链，实现从请求预处理到业务逻辑处理再到响应后处理的完整流程。