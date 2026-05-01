# springboot自动装配原理

Spring Boot自动装配（Auto-Configuration）是Spring Boot框架的核心特性之一，它能够根据项目中的依赖和配置自动配置Spring应用程序。让我详细解释一下它的工作原理：

## 1. 核心机制

### @SpringBootApplication注解

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication`是一个组合注解，包含了：

- `@SpringBootConfiguration`：标识这是一个配置类
- `@EnableAutoConfiguration`：启用自动装配
- `@ComponentScan`：启用组件扫描

### @EnableAutoConfiguration的作用

这个注解通过`@Import(AutoConfigurationImportSelector.class)`导入了自动装配的选择器。

## 2. 自动装配的执行流程

### 第一步：加载自动装配类

```java
// AutoConfigurationImportSelector的核心方法
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
        EnableAutoConfiguration.class, getBeanClassLoader());
    return configurations;
}
```

这个方法会读取所有jar包中的`META-INF/spring.factories`文件，找到`EnableAutoConfiguration`对应的配置类。

### 第二步：条件筛选

每个自动装配类都会使用条件注解进行筛选：

```java
@Configuration
@ConditionalOnClass(DataSource.class)  // 类路径下存在DataSource类
@ConditionalOnMissingBean(DataSource.class)  // 容器中没有DataSource bean
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    // 配置逻辑
}
```

### 第三步：实例化和注册Bean

满足条件的配置类会被实例化，其中定义的Bean会被注册到Spring容器中。

## 3. 关键组件详解

### SpringFactoriesLoader

负责加载`META-INF/spring.factories`文件中的配置：

```properties
# spring.factories文件示例
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### 条件注解（Conditional Annotations）

常用的条件注解：

- `@ConditionalOnClass`：类路径下存在指定类
- `@ConditionalOnMissingClass`：类路径下不存在指定类
- `@ConditionalOnBean`：容器中存在指定Bean
- `@ConditionalOnMissingBean`：容器中不存在指定Bean
- `@ConditionalOnProperty`：配置文件中存在指定属性

### ConfigurationProperties

用于绑定外部配置：

```java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {
    private String url;
    private String username;
    private String password;
    // getter/setter
}
```

## 4. 实际应用示例

### 数据库自动装配

当你在项目中引入数据库相关依赖时：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

Spring Boot会自动：

1. 检测到类路径下有JPA和MySQL驱动
2. 自动配置DataSource、EntityManagerFactory、TransactionManager等
3. 根据application.properties中的配置创建数据库连接

### Web应用自动装配

引入web starter：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

自动装配会：

1. 配置嵌入式Tomcat服务器
2. 配置Spring MVC的DispatcherServlet
3. 配置默认的错误处理、静态资源处理等

## 5. 自定义自动装配

### 创建自定义starter

```java
@Configuration
@ConditionalOnClass(MyService.class)
@ConditionalOnProperty(prefix = "myservice", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
}
```

在`META-INF/spring.factories`中注册：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyServiceAutoConfiguration
```

## 6. 优势与注意事项

### 优势

- **约定优于配置**：减少了大量的XML配置
- **快速开发**：开箱即用的功能
- **智能化**：根据依赖自动推断配置需求

### 注意事项

- 理解自动装配的执行顺序，避免循环依赖
- 合理使用`@ConditionalOnMissingBean`避免配置冲突
- 必要时可以通过`@EnableAutoConfiguration(exclude = ...)`排除某些自动装配

## 扩展知识

### 启动优化

- 使用`spring.factories`的`org.springframework.context.ApplicationListener`可以监听启动过程
- 通过`@AutoConfigureOrder`控制自动装配的顺序
- 使用`@ConditionalOnWebApplication`区分Web和非Web应用

### 调试技巧

启用调试模式查看自动装配报告：

```properties
debug=true
```

这会输出哪些配置被应用，哪些被跳过以及跳过的原因。

Spring Boot的自动装配机制大大简化了Spring应用的配置，让开发者能够专注于业务逻辑而不是繁琐的配置工作。