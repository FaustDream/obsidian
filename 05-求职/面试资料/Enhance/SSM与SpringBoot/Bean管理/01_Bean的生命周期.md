# Bean的生命周期

Bean的生命周期是Spring框架的核心概念之一，它描述了从Bean的创建到销毁的完整过程。让我详细为您介绍：

## Bean生命周期的完整流程

### 1. 实例化阶段

- **BeanDefinition加载**：Spring容器首先读取配置文件或注解，创建BeanDefinition对象
- **实例化Bean**：使用反射机制调用构造器创建Bean实例

### 2. 属性赋值阶段

- **依赖注入**：Spring通过setter方法或字段注入，为Bean的属性赋值
- **循环依赖处理**：如果存在循环依赖，Spring会通过三级缓存机制解决

### 3. 初始化前置处理

- **Aware接口回调**：如果Bean实现了BeanNameAware、BeanFactoryAware等接口，Spring会调用相应的set方法
- **BeanPostProcessor前置处理**：执行所有BeanPostProcessor的postProcessBeforeInitialization方法

### 4. 初始化阶段

- **@PostConstruct注解方法**：执行标注了@PostConstruct的方法
- **InitializingBean接口**：如果实现了InitializingBean接口，调用afterPropertiesSet方法
- **自定义初始化方法**：执行@Bean注解中指定的initMethod或XML配置的init-method

### 5. 初始化后置处理

- **BeanPostProcessor后置处理**：执行postProcessAfterInitialization方法
- **AOP代理创建**：如果需要，在此阶段创建代理对象

### 6. 使用阶段

- Bean已完全初始化，可以被应用程序使用

### 7. 销毁阶段

- **@PreDestroy注解方法**：执行标注了@PreDestroy的方法
- **DisposableBean接口**：如果实现了DisposableBean接口，调用destroy方法
- **自定义销毁方法**：执行destroyMethod指定的方法

## 代码示例

```java
@Component
public class UserService implements BeanNameAware, InitializingBean, DisposableBean {
    
    private String beanName;
    
    // 构造器
    public UserService() {
        System.out.println("1. 构造器执行");
    }
    
    // Aware接口回调
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("2. BeanNameAware.setBeanName执行");
    }
    
    // 初始化方法
    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct执行");
    }
    
    @Override
    public void afterPropertiesSet() {
        System.out.println("4. InitializingBean.afterPropertiesSet执行");
    }
    
    // 销毁方法
    @PreDestroy
    public void preDestroy() {
        System.out.println("5. @PreDestroy执行");
    }
    
    @Override
    public void destroy() {
        System.out.println("6. DisposableBean.destroy执行");
    }
}
```

## 扩展知识点

### BeanPostProcessor的应用

BeanPostProcessor是Spring提供的扩展点，允许我们在Bean初始化前后进行自定义处理：

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("前置处理: " + beanName);
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("后置处理: " + beanName);
        return bean;
    }
}
```

### 循环依赖处理机制

Spring通过三级缓存解决循环依赖：

- **一级缓存**：singletonObjects，存放完全初始化的Bean
- **二级缓存**：earlySingletonObjects，存放原始Bean对象
- **三级缓存**：singletonFactories，存放Bean工厂对象

### 不同作用域的生命周期差异

- **Singleton**：容器启动时创建，容器关闭时销毁
- **Prototype**：每次getBean时创建新实例，Spring不管理销毁
- **Request/Session**：在Web环境中，随请求/会话创建和销毁

理解Bean生命周期对于Spring开发至关重要，它不仅帮助我们理解Spring的工作原理，还能让我们在合适的时机进行自定义处理，提高应用程序的灵活性和可维护性。