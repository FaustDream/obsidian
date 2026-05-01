`@Lookup` 是 Spring Framework 中的一个注解，用于方法级别的依赖注入，它允许你在运行时动态获取 Bean 实例。这在处理单例 Bean 需要获取原型 Bean 的场景中特别有用。

## 基本概念

`@Lookup` 注解主要解决单例 Bean 中需要多次获取原型（prototype）Bean 的问题。由于单例 Bean 只创建一次，如果直接注入原型 Bean，那么每次使用的都是同一个实例，违背了原型 Bean 的设计初衷。

## 使用方式

### 1. 基本用法

```java
@Component
public class SingletonBean {
    
    // 这个方法会被 Spring 在运行时重写
    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null; // 返回值会被 Spring 覆盖
    }
    
    public void doSomething() {
        // 每次调用都会获得新的原型实例
        PrototypeBean prototype = getPrototypeBean();
        prototype.performAction();
    }
}
```

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    public void performAction() {
        System.out.println("执行操作: " + this.hashCode());
    }
}
```

### 2. 指定 Bean 名称

```java
@Component
public class SingletonBean {
    
    @Lookup("myPrototypeBean")
    public PrototypeBean getSpecificBean() {
        return null;
    }
}
```

### 3. 抽象方法使用

```java
@Component
public abstract class AbstractSingletonBean {
    
    @Lookup
    public abstract PrototypeBean createPrototypeBean();
    
    public void processData() {
        PrototypeBean bean = createPrototypeBean();
        // 使用新创建的原型实例
    }
}
```

## 实现原理

Spring 使用 CGLIB 或 JDK 动态代理来实现 `@Lookup` 功能：

1. Spring 会为包含 `@Lookup` 方法的类创建子类或代理
2. 重写标注了 `@Lookup` 的方法
3. 在运行时，这些方法会委托给 `ApplicationContext.getBean()` 来获取实例

## 使用场景

### 1. 单例中使用原型 Bean

```java
@Service
public class OrderService {
    
    @Lookup
    public OrderProcessor createOrderProcessor() {
        return null;
    }
    
    public void processOrder(Order order) {
        // 每个订单使用新的处理器实例
        OrderProcessor processor = createOrderProcessor();
        processor.process(order);
    }
}

@Component
@Scope("prototype")
public class OrderProcessor {
    private String processingId = UUID.randomUUID().toString();
    
    public void process(Order order) {
        System.out.println("处理订单: " + order.getId() + 
                         ", 处理器ID: " + processingId);
    }
}
```

### 2. 工厂模式实现

```java
@Component
public class TaskFactory {
    
    @Lookup("emailTask")
    public Task createEmailTask() {
        return null;
    }
    
    @Lookup("smsTask")  
    public Task createSmsTask() {
        return null;
    }
    
    public Task createTask(String type) {
        switch (type) {
            case "email":
                return createEmailTask();
            case "sms":
                return createSmsTask();
            default:
                throw new IllegalArgumentException("未知任务类型: " + type);
        }
    }
}
```

## 注意事项

1. **方法限制**：`@Lookup` 方法不能是 `private` 或 `final`
2. **返回值**：方法体中的返回值会被忽略，Spring 会提供实际的 Bean 实例
3. **性能考虑**：每次调用都会从容器中获取 Bean，有一定性能开销
4. **测试友好性**：可以通过继承来覆盖 `@Lookup` 方法，便于单元测试

## 替代方案

### 1. ApplicationContextAware

```java
@Component
public class SingletonBean implements ApplicationContextAware {
    
    private ApplicationContext applicationContext;
    
    @Override
    public void setApplicationContext(ApplicationContext context) {
        this.applicationContext = context;
    }
    
    public PrototypeBean getPrototypeBean() {
        return applicationContext.getBean(PrototypeBean.class);
    }
}
```

### 2. ObjectFactory

```java
@Component
public class SingletonBean {
    
    @Autowired
    private ObjectFactory<PrototypeBean> prototypeBeanFactory;
    
    public void doSomething() {
        PrototypeBean bean = prototypeBeanFactory.getObject();
        bean.performAction();
    }
}
```

`@Lookup` 提供了一种优雅的方式来处理不同作用域 Bean 之间的依赖关系，特别适合需要在单例 Bean 中动态获取原型 Bean 的场景。