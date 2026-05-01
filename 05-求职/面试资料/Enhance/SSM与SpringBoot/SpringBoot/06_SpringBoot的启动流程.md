# SpringBoot的启动流程

SpringBoot的启动流程可以分为以下几个核心阶段：

## 1. 应用启动入口

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

启动从`SpringApplication.run()`方法开始，这是整个流程的入口点。

## 2. SpringApplication对象创建

在`run()`方法内部，首先会创建`SpringApplication`实例：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return new SpringApplication(primarySource).run(args);
}
```

创建过程中会进行：

- **应用类型推断**：判断是SERVLET、REACTIVE还是NONE类型
- **初始化器加载**：从`META-INF/spring.factories`加载`ApplicationContextInitializer`
- **监听器加载**：加载`ApplicationListener`实现类
- **主类推断**：通过堆栈信息确定main方法所在的类

## 3. 运行准备阶段

调用`run(args)`方法后：

### 3.1 启动监听器

```java
SpringApplicationRunListeners listeners = getRunListeners(args);
listeners.starting(); // 发布ApplicationStartingEvent事件
```

### 3.2 环境准备

```java
ConfigurableEnvironment environment = prepareEnvironment(listeners, args);
```

- 创建环境对象（StandardServletEnvironment等）
- 配置环境（处理命令行参数、系统属性等）
- 发布`ApplicationEnvironmentPreparedEvent`事件

### 3.3 打印Banner

```java
Banner printedBanner = printBanner(environment);
```

## 4. 应用上下文创建

```java
context = createApplicationContext();
```

根据应用类型创建对应的ApplicationContext：

- **SERVLET**：`AnnotationConfigServletWebServerApplicationContext`
- **REACTIVE**：`AnnotationConfigReactiveWebServerApplicationContext`
- **NONE**：`AnnotationConfigApplicationContext`

## 5. 上下文准备阶段

```java
prepareContext(context, environment, listeners, args, printedBanner);
```

这个阶段会：

- 应用初始化器到上下文
- 发布`ApplicationContextInitializedEvent`事件
- 注册启动参数为Bean
- 加载启动类到BeanDefinitionRegistry
- 发布`ApplicationPreparedEvent`事件

## 6. 刷新上下文（核心阶段）

```java
refreshContext(context);
```

这是Spring容器初始化的核心流程，基于AbstractApplicationContext的refresh()方法：

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 1. 准备刷新
        prepareRefresh();
        
        // 2. 获取BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 3. 准备BeanFactory
        prepareBeanFactory(beanFactory);
        
        try {
            // 4. 后置处理BeanFactory
            postProcessBeanFactory(beanFactory);
            
            // 5. 调用BeanFactory后置处理器
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 6. 注册Bean后置处理器
            registerBeanPostProcessors(beanFactory);
            
            // 7. 初始化消息源
            initMessageSource();
            
            // 8. 初始化应用事件多播器
            initApplicationEventMulticaster();
            
            // 9. 刷新特定上下文（启动内嵌服务器）
            onRefresh();
            
            // 10. 注册监听器
            registerListeners();
            
            // 11. 实例化所有剩余的单例Bean
            finishBeanFactoryInitialization(beanFactory);
            
            // 12. 完成刷新
            finishRefresh();
        }
        catch (BeansException ex) {
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        }
    }
}
```

## 7. 刷新后处理

```java
afterRefresh(context, args);
```

## 8. 启动完成

- 发布`ApplicationStartedEvent`事件
- 调用`ApplicationRunner`和`CommandLineRunner`
- 发布`ApplicationReadyEvent`事件

## 关键技术点

### 自动配置机制

通过`@EnableAutoConfiguration`注解，SpringBoot会：

1. 扫描classpath下的`META-INF/spring.factories`
2. 加载`EnableAutoConfiguration`对应的配置类
3. 根据条件注解（`@ConditionalOnClass`、`@ConditionalOnMissingBean`等）决定是否生效

### 内嵌服务器启动

在`onRefresh()`阶段，SpringBoot会：

1. 创建内嵌服务器（Tomcat、Jetty、Undertow）
2. 配置服务器参数
3. 启动服务器并监听端口

### 事件驱动机制

整个启动过程通过事件驱动：

- `ApplicationStartingEvent`
- `ApplicationEnvironmentPreparedEvent`
- `ApplicationContextInitializedEvent`
- `ApplicationPreparedEvent`
- `ApplicationStartedEvent`
- `ApplicationReadyEvent`

## 扩展知识

### 启动性能优化

1. **延迟初始化**：`spring.main.lazy-initialization=true`
2. **排除不必要的自动配置**：`@EnableAutoConfiguration(exclude = {...})`
3. **使用ApplicationContextInitializer**进行早期初始化
4. **Profile优化**：针对不同环境使用不同配置

### 故障排查

1. **启动日志分析**：通过`--debug`参数查看自动配置报告
2. **Actuator端点**：`/actuator/beans`、`/actuator/conditions`等
3. **事件监听**：实现`ApplicationListener`监听启动事件

这个启动流程展现了SpringBoot"约定优于配置"的设计理念，通过自动配置和智能默认值，大大简化了Spring应用的启动过程。