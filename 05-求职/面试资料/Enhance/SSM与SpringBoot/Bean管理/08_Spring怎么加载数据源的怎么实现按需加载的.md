# Spring怎么加载数据源的？怎么实现按需加载的？

Spring的数据源加载机制是一个多层次的过程，我来详细解释一下：

## Spring数据源加载机制

### 1. 自动配置加载

Spring Boot通过`DataSourceAutoConfiguration`自动配置类来加载数据源：

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

### 2. 配置属性绑定

通过`DataSourceProperties`类绑定配置文件中的数据源属性：

```java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {
    private String url;
    private String username;
    private String password;
    private String driverClassName;
    // getter/setter...
}
```

### 3. 数据源创建流程

Spring按以下优先级创建数据源：

1. 检查是否存在自定义DataSource Bean
2. 使用配置的连接池类型（HikariCP、Tomcat、Commons DBCP2等）
3. 创建并配置数据源实例

## 按需加载的实现方式

### 1. 懒加载配置

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Lazy // 懒加载注解
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/db")
            .username("user")
            .password("password")
            .build();
    }
}
```

### 2. 条件化加载

```java
@Configuration
public class ConditionalDataSourceConfig {
    
    @Bean
    @ConditionalOnProperty(name = "app.datasource.enabled", havingValue = "true")
    public DataSource conditionalDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

### 3. 动态数据源切换

```java
public class DynamicDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDataSourceType();
    }
    
    // 按需加载不同的数据源
    public void addDataSource(String key, DataSource dataSource) {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(key, dataSource);
        setTargetDataSources(targetDataSources);
        afterPropertiesSet();
    }
}
```

### 4. 基于Profile的按需加载

```java
@Configuration
@Profile("dev")
public class DevDataSourceConfig {
    @Bean
    public DataSource devDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:h2:mem:testdb")
            .build();
    }
}

@Configuration
@Profile("prod")
public class ProdDataSourceConfig {
    @Bean
    public DataSource prodDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://prod-server:3306/db")
            .build();
    }
}
```

## 高级按需加载实现

### 1. 自定义数据源工厂

```java
@Component
public class DataSourceFactory {
    
    private final Map<String, DataSource> dataSources = new ConcurrentHashMap<>();
    
    public DataSource getDataSource(String key) {
        return dataSources.computeIfAbsent(key, this::createDataSource);
    }
    
    private DataSource createDataSource(String key) {
        // 根据key动态创建数据源
        return DataSourceBuilder.create()
            .url(getUrlByKey(key))
            .build();
    }
}
```

### 2. 基于注解的按需加载

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DataSourceSelector {
    String value() default "default";
}

@Aspect
@Component
public class DataSourceAspect {
    
    @Around("@annotation(dataSourceSelector)")
    public Object around(ProceedingJoinPoint point, DataSourceSelector dataSourceSelector) throws Throwable {
        String dsKey = dataSourceSelector.value();
        DataSourceContextHolder.setDataSourceType(dsKey);
        try {
            return point.proceed();
        } finally {
            DataSourceContextHolder.clear();
        }
    }
}
```

## 相关扩展知识

### 1. 连接池按需初始化

```java
@Configuration
public class LazyConnectionPoolConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariConfig hikariConfig() {
        HikariConfig config = new HikariConfig();
        config.setInitializationFailTimeout(-1); // 延迟初始化
        return config;
    }
}
```

### 2. 多数据源事务管理

```java
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    
    @Bean
    public PlatformTransactionManager transactionManager(
            @Qualifier("primaryDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

### 3. 数据源健康检查

```java
@Component
public class DataSourceHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        try {
            // 按需检查数据源连接状态
            return Health.up().build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

这种按需加载机制的优势在于：减少启动时间、节省资源、提高系统灵活性，特别适用于微服务架构和多租户系统。