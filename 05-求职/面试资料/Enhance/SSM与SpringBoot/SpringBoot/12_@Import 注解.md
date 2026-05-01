`@Import` 注解是 Spring 框架中一个非常重要的配置注解，它的主要作用是**将其他配置类导入到当前配置类中**，实现配置的模块化和组合。

## 主要作用

### 1. 导入配置类

`@Import` 可以将一个或多个配置类导入到当前配置类中，使得被导入的配置类中定义的 Bean 能够被 Spring 容器识别和管理。

```java
@Configuration
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() {
        // 数据源配置
        return new HikariDataSource();
    }
}

@Configuration
@Import(DatabaseConfig.class)  // 导入数据库配置
public class AppConfig {
    // 当前配置类的其他Bean定义
}
```

### 2. 导入普通类

`@Import` 也可以导入普通的类（非@Configuration注解的类），Spring会将这些类注册为Bean。

```java
public class UserService {
    // 普通类
}

@Configuration
@Import(UserService.class)  // 导入普通类
public class AppConfig {
}
```

### 3. 导入ImportSelector实现类

通过实现 `ImportSelector` 接口，可以动态决定要导入哪些类。

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 动态决定要导入的类
        return new String[]{"com.example.ServiceA", "com.example.ServiceB"};
    }
}

@Configuration
@Import(MyImportSelector.class)
public class AppConfig {
}
```

### 4. 导入ImportBeanDefinitionRegistrar实现类

通过实现 `ImportBeanDefinitionRegistrar` 接口，可以手动注册Bean定义。

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, 
                                      BeanDefinitionRegistry registry) {
        // 手动注册Bean定义
        BeanDefinition beanDefinition = new RootBeanDefinition(UserService.class);
        registry.registerBeanDefinition("userService", beanDefinition);
    }
}

@Configuration
@Import(MyImportBeanDefinitionRegistrar.class)
public class AppConfig {
}
```

## 使用场景

### 1. 模块化配置

将不同功能模块的配置分离，通过 `@Import` 进行组合：

```java
@Configuration
public class SecurityConfig {
    // 安全相关配置
}

@Configuration  
public class WebConfig {
    // Web相关配置
}

@Configuration
@Import({SecurityConfig.class, WebConfig.class})
public class MainConfig {
    // 主配置类
}
```

### 2. 第三方库集成

在集成第三方库时，通过 `@Import` 导入第三方提供的配置类：

```java
@Configuration
@Import({ThirdPartyConfig.class})
public class AppConfig {
}
```

### 3. 自动配置

Spring Boot 中大量使用了 `@Import` 来实现自动配置：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

## 与其他注解的区别

|注解|作用|适用场景|
|---|---|---|
|`@ComponentScan`|扫描包路径下的组件|自动发现和注册Bean|
|`@Import`|显式导入指定的类或配置|精确控制要导入的组件|
|`@Configuration`|标识配置类|定义Bean的配置类|

## 扩展知识

### 1. @Enable* 注解族

Spring 中很多 `@Enable*` 注解都是基于 `@Import` 实现的：

```java
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {
}

@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {
}
```

### 2. 条件化导入

结合 `@Conditional` 注解，可以实现条件化的导入：

```java
@Configuration
@Import(DatabaseConfig.class)
@ConditionalOnProperty(name = "database.enabled", havingValue = "true")
public class ConditionalConfig {
}
```

### 3. 导入顺序

可以通过 `@Order` 注解控制导入的顺序：

```java
@Configuration
@Order(1)
public class FirstConfig {
}

@Configuration  
@Order(2)
public class SecondConfig {
}

@Configuration
@Import({FirstConfig.class, SecondConfig.class})
public class MainConfig {
}
```

`@Import` 注解为 Spring 提供了灵活的配置组合机制，是理解 Spring 配置体系和自动配置原理的重要知识点。