单例Bean的循环依赖问题是Spring框架中一个重要的概念，指的是两个或多个单例Bean在创建过程中相互依赖，形成循环引用的情况。

## 什么是循环依赖

循环依赖发生在以下场景：

- Bean A 依赖 Bean B
- Bean B 依赖 Bean A
- 或者形成更长的依赖链：A → B → C → A

## 典型示例

```java
@Component
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}

@Component  
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
}
```

## Spring如何解决循环依赖

Spring通过**三级缓存机制**解决单例Bean的循环依赖：

1. **一级缓存（singletonObjects）**：存放完全初始化好的Bean
2. **二级缓存（earlySingletonObjects）**：存放原始的Bean对象（未填充属性）
3. **三级缓存（singletonFactories）**：存放Bean工厂对象

### 解决过程：

1. 创建Bean A时，先将A的工厂对象放入三级缓存
2. 填充A的属性时发现需要Bean B
3. 创建Bean B，同样将B的工厂对象放入三级缓存
4. 填充B的属性时需要Bean A
5. 从三级缓存获取A的工厂，创建A的早期引用放入二级缓存
6. B完成创建，放入一级缓存
7. A获得B的引用，完成创建，放入一级缓存

## 无法解决的情况

Spring无法解决以下循环依赖：

- **构造器循环依赖**：==因为构造器注入发生在Bean实例化阶段，此时还没有半成品Bean可以提前暴露==
- **prototype作用域的循环依赖**：prototype Bean每次都创建新实例，无法使用缓存机制
- **@Async注解的循环依赖**：异步方法需要代理对象，代理创建时机与循环依赖解决机制冲突

## 最佳实践

1. **重新设计**：通过重构消除循环依赖
2. **使用@Lazy注解**：延迟Bean的初始化
3. **setter注入**：避免构造器循环依赖
4. **事件驱动**：通过Spring事件机制解耦

循环依赖虽然Spring能够解决，但它通常暗示设计上的问题，应该尽量避免。