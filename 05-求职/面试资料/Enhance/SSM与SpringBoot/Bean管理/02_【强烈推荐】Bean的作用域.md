# Bean的作用域

Bean的作用域(Scope)是Spring框架中的核心概念，它决定了Spring容器如何创建和管理Bean实例的生命周期。

## 主要作用域类型

**1. singleton（单例）- 默认作用域**

- 整个Spring容器中只创建一个Bean实例
- 所有对该Bean的请求都返回同一个实例
- 容器启动时创建，容器销毁时销毁

```java
@Component
@Scope("singleton") // 或者省略，因为是默认值
public class UserService {
    // 同一个实例被所有调用者共享
}
```

**2. prototype（原型）**

- 每次请求都创建新的Bean实例
- Spring容器不负责prototype Bean的完整生命周期管理
- 适用于有状态的Bean

```java
@Component
@Scope("prototype")
public class ShoppingCart {
    // 每次注入都是新实例
}
```

**3. Web相关作用域（需要Web环境）**

**request作用域**

- 每个HTTP请求创建一个Bean实例
- 请求结束时销毁

```java
@Component
@Scope("request")
public class RequestContext {
    // 每个HTTP请求都有独立实例
}
```

**session作用域**

- 每个HTTP Session创建一个Bean实例
- Session失效时销毁

```java
@Component
@Scope("session")
public class UserSession {
    // 每个用户会话独立实例
}
```

**application作用域**

- 整个ServletContext生命周期内只有一个实例
- 类似singleton，但作用范围是ServletContext

## 配置方式

**注解方式：**

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class MyBean { }
```

**XML配置：**

```xml
<bean id="myBean" class="com.example.MyBean" scope="prototype"/>
```

**Java配置：**

```java
@Configuration
public class AppConfig {
    @Bean
    @Scope("prototype")
    public MyBean myBean() {
        return new MyBean();
    }
}
```

## 实际应用场景

**singleton适用于：**

- 无状态的Service层组件
- DAO层组件
- 工具类Bean

**prototype适用于：**

- 有状态的对象
- 每次使用都需要不同配置的对象
- 重量级对象的按需创建

## 注意事项

**1. singleton中注入prototype的问题**
> SingletonService在容器启动时只会被创建一次，此时注入的PrototypeBean实例也只会创建一次，后续每次调用都是同一个实例，失去了原型的意义。

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    public void process() {
        System.out.println("Processing: " + this.hashCode());
    }
}

@Service
public class SingletonService {
    @Autowired
    private PrototypeBean prototypeBean; // 这里有问题：注入的始终是同一个实例
    
    public void doSomething() {
        prototypeBean.process();
    }
}
```

**解决方案：**

```java
// 方案1：使用@Lookup注解
@Component
public class SingletonService {
    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null; // Spring会重写这个方法
    }
}

// 方案2：使用ApplicationContext
@Component
public class SingletonService {
    @Autowired
    private ApplicationContext context;
    
    public PrototypeBean getPrototypeBean() {
        return context.getBean(PrototypeBean.class);
    }
}
```

**2. 线程安全问题**

- singleton Bean需要考虑线程安全
- prototype Bean每次都是新实例，天然线程安全

## 扩展知识

**自定义作用域：** 可以实现`Scope`接口来创建自定义作用域：

```java
public class ThreadScope implements Scope {
    private final ThreadLocal<Map<String, Object>> threadLocal = 
        new ThreadLocal<Map<String, Object>>() {
            @Override
            protected Map<String, Object> initialValue() {
                return new HashMap<>();
            }
        };
    
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadLocal.get();
        Object obj = scope.get(name);
        if (obj == null) {
            obj = objectFactory.getObject();
            scope.put(name, obj);
        }
        return obj;
    }
    // 其他方法实现...
}
```

理解Bean作用域对于合理设计Spring应用架构、避免内存泄漏和线程安全问题都非常重要。