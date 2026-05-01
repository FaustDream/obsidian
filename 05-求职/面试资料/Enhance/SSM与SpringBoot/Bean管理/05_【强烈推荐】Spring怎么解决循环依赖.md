# Spring怎么解决循环依赖

Spring解决循环依赖主要通过**三级缓存机制**来实现，这是一个经典的设计模式应用。

## 什么是循环依赖

循环依赖指两个或多个Bean相互引用，形成闭环：

```java
@Component
public class A {
    @Autowired
    private B b;
}

@Component  
public class B {
    @Autowired
    private A a;
}
```

## Spring的三级缓存机制

Spring使用三个Map来解决循环依赖：

```java
// 一级缓存：完整的Bean实例
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// 二级缓存：早期Bean实例（未完成依赖注入）
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// 三级缓存：Bean工厂，用于创建早期Bean实例
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

## 解决过程详解

以A依赖B、B依赖A为例：

1. **创建A实例**：实例化A（调用构造函数），放入三级缓存
2. **A依赖注入**：发现需要B，开始创建B
3. **创建B实例**：实例化B（调用构造函数），放入三级缓存
4. **B依赖注入**：发现需要A，从三级缓存获取A的早期实例并放入二级缓存
5. **B完成创建**：B依赖注入完成，放入一级缓存
6. **A完成创建**：A获得完整的B实例，完成依赖注入，放入一级缓存

## 核心源码逻辑

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 1. 先从一级缓存获取完整Bean
    Object singletonObject = this.singletonObjects.get(beanName);
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 2. 从二级缓存获取早期Bean
            singletonObject = this.earlySingletonObjects.get(beanName);
            
            if (singletonObject == null && allowEarlyReference) {
                // 3. 从三级缓存获取Bean工厂
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    // 放入二级缓存，移除三级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

## 解决范围与限制

**能解决的情况**：

- 单例Bean的setter注入循环依赖
- 字段注入（@Autowired）循环依赖

**不能解决的情况**：

- ==构造器注入循环依赖（因为实例化都无法完成）==
- 多例Bean循环依赖（每次都创建新实例）
- 异步初始化的Bean循环依赖

## 为什么需要三级缓存

**二级缓存不够的原因**：

- 支持AOP代理：如果Bean需要被代理，三级缓存中的ObjectFactory可以返回代理对象
- 延迟代理创建：只有真正需要时才创建代理，提高性能

## 最佳实践建议

1. **避免循环依赖**：通过重构代码结构，使用中介者模式等
2. **使用@Lazy注解**：==延迟初始化，打破循环==
3. **构造器注入改为setter注入**：允许Spring的三级缓存机制生效
4. **使用ApplicationContext**：手动获取Bean，避免直接依赖

这种设计体现了Spring IoC容器的强大之处，通过巧妙的缓存机制解决了复杂的循环依赖问题。