客户端发送http请求，DispatcherServlet(前端控制器)接收到请求，HandlerMapping(处理器映射器)根据请求的url能找到是哪个controller的方法然后返回给dispatcherServlet，HandlerAdapter(处理器适配器)就来调用找到的处理器方法，然后controller处理请求返回给dispatcherServlet，前后端不分离的情况，前端控制器会再调视图解析器，返回一个modelView交给页面进行最后的渲染。

-----
-----


Spring MVC的执行流程是一个经典的面试题，我来详细解释一下整个请求处理的流程：

## Spring MVC执行流程（8个核心步骤）

### 1. 用户请求到达DispatcherServlet

- 客户端发送HTTP请求到Web服务器
- **DispatcherServlet**（前端控制器）接收所有请求，它是Spring MVC的核心入口

### 2. DispatcherServlet查询HandlerMapping

- DispatcherServlet调用**HandlerMapping**（处理器映射器）
- HandlerMapping根据请求URL找到对应的Handler（通常是Controller中的方法）
- ==返回HandlerExecutionChain（包含Handler和拦截器链）==

### 3. DispatcherServlet获取HandlerAdapter

- DispatcherServlet通过Handler找到对应的**HandlerAdapter**（处理器适配器）
- HandlerAdapter的作用是调用具体的Handler方法

### 4. HandlerAdapter执行Handler

- HandlerAdapter调用具体的Controller方法
- 在调用前后会执行拦截器的preHandle和postHandle方法

### 5. Handler执行业务逻辑并返回ModelAndView

- Controller执行具体的业务逻辑
- 返回**ModelAndView**对象（包含模型数据和视图名称）

### 6. DispatcherServlet查询ViewResolver

- DispatcherServlet调用**ViewResolver**（视图解析器）
- ViewResolver根据视图名称解析出具体的View对象

### 7. DispatcherServlet渲染视图

- DispatcherServlet调用View的render方法
- View将Model数据渲染到具体的视图中（如JSP、Thymeleaf等）

### 8. 返回响应给用户

- 将渲染好的视图作为HTTP响应返回给客户端

## 核心组件说明

### DispatcherServlet（前端控制器）

```java
// 配置示例
@EnableWebMvc
@Configuration
public class WebConfig implements WebMvcConfigurer {
    // 配置视图解析器、拦截器等
}
```

### HandlerMapping（处理器映射器）

- **RequestMappingHandlerMapping**：处理@RequestMapping注解
- **BeanNameUrlHandlerMapping**：根据bean名称映射URL

### HandlerAdapter（处理器适配器）

- **RequestMappingHandlerAdapter**：处理@RequestMapping标注的方法
- **HttpRequestHandlerAdapter**：处理HttpRequestHandler
- **SimpleControllerHandlerAdapter**：处理Controller接口

### ViewResolver（视图解析器）

```java
@Bean
public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
}
```

## 扩展知识点

### 1. 拦截器执行时机

```java
public class MyInterceptor implements HandlerInterceptor {
    // 在Handler执行前调用
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) { }
    
    // 在Handler执行后，视图渲染前调用
    public void postHandle(HttpServletRequest request, 
                          HttpServletResponse response, 
                          Object handler, 
                          ModelAndView modelAndView) { }
    
    // 在整个请求完成后调用
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, Exception ex) { }
}
```

### 2. 异常处理机制

- **@ExceptionHandler**：方法级异常处理
- **@ControllerAdvice**：全局异常处理
- **HandlerExceptionResolver**：异常解析器

### 3. RESTful API处理

```java
@RestController
public class ApiController {
    @GetMapping("/api/users/{id}")
    public User getUser(@PathVariable Long id) {
        // 直接返回对象，会被转换为JSON
        return userService.findById(id);
    }
}
```

### 4. 与Spring Boot的关系

在Spring Boot中，很多配置都是自动完成的：

- 自动配置DispatcherServlet
- 自动注册HandlerMapping和HandlerAdapter
- 提供默认的错误页面处理

这个流程理解透彻后，有助于排查Spring MVC相关问题，也是理解Spring框架设计模式的好例子。