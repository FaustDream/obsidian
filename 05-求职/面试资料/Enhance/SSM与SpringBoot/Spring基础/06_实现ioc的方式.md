# 实现ioc的方式

IoC（控制反转）是一种设计原则，有多种实现方式。让我为你详细介绍几种主要的实现方式：

## 1. 依赖注入（Dependency Injection）

### 构造器注入

```java
public class UserService {
    private final UserRepository userRepository;
    
    // 通过构造器注入依赖
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Setter注入

```java
public class UserService {
    private UserRepository userRepository;
    
    // 通过setter方法注入依赖
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 接口注入

```java
public interface UserRepositoryAware {
    void setUserRepository(UserRepository userRepository);
}

public class UserService implements UserRepositoryAware {
    private UserRepository userRepository;
    
    @Override
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## 2. 服务定位器模式（Service Locator）

```java
public class ServiceLocator {
    private static Map<Class<?>, Object> services = new HashMap<>();
    
    public static <T> T getService(Class<T> serviceClass) {
        return (T) services.get(serviceClass);
    }
    
    public static void registerService(Class<?> serviceClass, Object service) {
        services.put(serviceClass, service);
    }
}

// 使用方式
public class UserService {
    private UserRepository userRepository;
    
    public UserService() {
        // 主动从服务定位器获取依赖
        this.userRepository = ServiceLocator.getService(UserRepository.class);
    }
}
```

## 3. 注解驱动的依赖注入

```java
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
}
```

## 4. 工厂模式结合IoC

```java
public class ServiceFactory {
    private static final Map<Class<?>, Object> instances = new HashMap<>();
    
    @SuppressWarnings("unchecked")
    public static <T> T getInstance(Class<T> clazz) {
        if (!instances.containsKey(clazz)) {
            try {
                T instance = clazz.getDeclaredConstructor().newInstance();
                // 注入依赖
                injectDependencies(instance);
                instances.put(clazz, instance);
            } catch (Exception e) {
                throw new RuntimeException("Failed to create instance", e);
            }
        }
        return (T) instances.get(clazz);
    }
    
    private static void injectDependencies(Object instance) {
        // 通过反射注入依赖
        Field[] fields = instance.getClass().getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(Inject.class)) {
                field.setAccessible(true);
                try {
                    field.set(instance, getInstance(field.getType()));
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("Failed to inject dependency", e);
                }
            }
        }
    }
}
```

## 5. 事件驱动的IoC

```java
public class EventDrivenContainer {
    private final Map<Class<?>, Object> beans = new HashMap<>();
    private final List<BeanPostProcessor> postProcessors = new ArrayList<>();
    
    public void registerBean(String name, Object bean) {
        beans.put(bean.getClass(), bean);
        // 触发后处理器进行依赖注入
        postProcessors.forEach(processor -> processor.postProcessBean(bean));
    }
}
```

## 各种实现方式的比较

|实现方式|优点|缺点|适用场景|
|---|---|---|---|
|构造器注入|强制依赖、不可变、线程安全|构造器参数过多时复杂|必需依赖|
|Setter注入|灵活、可选依赖|可能导致不完整对象|可选依赖|
|服务定位器|简单易用|隐藏依赖关系、难以测试|遗留系统改造|
|注解驱动|代码简洁、配置少|依赖框架、编译时检查困难|现代应用开发|

## 相关知识拓展

**IoC容器的生命周期管理**：

- Bean的创建、初始化、销毁
- 单例模式vs原型模式
- 循环依赖的处理

**AOP与IoC的结合**：

- 通过动态代理实现横切关注点
- 在依赖注入时织入切面逻辑

**配置方式的演进**：

- XML配置 → 注解配置 → Java Config → 自动配置

这些实现方式在实际项目中经常结合使用，Spring框架就是这些模式的经典实现。