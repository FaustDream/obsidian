# 注解@Autowired和@Resource的区别？@Auowired是如何注入的？

我来详细解释一下这两个注解的区别以及@Autowired的注入机制。

## @Autowired和@Resource的区别

### 1. 来源不同

- **@Autowired**：Spring框架提供的注解，属于Spring特有的依赖注入注解
- **@Resource**：Java EE规范(JSR-250)提供的注解，是标准的Java注解

### 2. 注入方式不同

- **@Autowired**：默认按照**类型(byType)**进行注入
- **@Resource**：默认按照**名称(byName)**进行注入，如果找不到匹配的名称，再按类型注入

### 3. 使用示例对比

```java
// @Autowired 按类型注入
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // 按UserRepository类型查找bean
    
    @Autowired
    @Qualifier("userRepositoryImpl") // 需要配合@Qualifier指定具体实现
    private UserRepository specificRepo;
}

// @Resource 按名称注入
@Service
public class UserService {
    @Resource(name = "userRepositoryImpl")
    private UserRepository userRepository; // 按名称查找bean
    
    @Resource // 默认按属性名userRepository查找bean
    private UserRepository userRepository;
}
```

### 4. 属性配置差异

- **@Autowired**：只有`required`属性，默认为true
- **@Resource**：有`name`、`type`、`authenticationType`、`shareable`等属性

### 5. 适用范围

- **@Autowired**：可用于字段、方法、构造器、参数
- **@Resource**：主要用于字段和setter方法

## @Autowired的注入机制

### 1. 注入时机

@Autowired的注入发生在Spring容器的**bean实例化和属性填充阶段**，具体在`BeanPostProcessor`的处理过程中。

### 2. 核心处理器

主要由`AutowiredAnnotationBeanPostProcessor`负责处理：

```java
// 简化的处理流程
public class AutowiredAnnotationBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, 
            PropertyDescriptor[] pds, Object bean, String beanName) {
        // 1. 扫描@Autowired注解
        InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass());
        // 2. 执行注入
        metadata.inject(bean, beanName, pvs);
        return pvs;
    }
}
```

### 3. 详细注入流程

#### 第一步：扫描和缓存

```java
// Spring启动时扫描@Autowired注解
private InjectionMetadata buildAutowiringMetadata(Class<?> clazz) {
    List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
    
    // 扫描字段上的@Autowired
    ReflectionUtils.doWithLocalFields(clazz, field -> {
        if (field.isAnnotationPresent(Autowired.class)) {
            elements.add(new AutowiredFieldElement(field));
        }
    });
    
    // 扫描方法上的@Autowired
    ReflectionUtils.doWithLocalMethods(clazz, method -> {
        if (method.isAnnotationPresent(Autowired.class)) {
            elements.add(new AutowiredMethodElement(method));
        }
    });
    
    return new InjectionMetadata(clazz, elements);
}
```

#### 第二步：依赖查找

```java
// 根据类型查找依赖的bean
private Object resolveAutowiredDependency(DependencyDescriptor descriptor, 
        String beanName, Set<String> autowiredBeanNames, TypeConverter typeConverter) {
    
    // 1. 先从容器中按类型查找
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
    
    // 2. 如果找到多个候选者，进行筛选
    if (matchingBeans.size() > 1) {
        // 查找@Primary注解的bean
        String primaryBeanName = determinePrimaryCandidate(matchingBeans);
        if (primaryBeanName != null) {
            return matchingBeans.get(primaryBeanName);
        }
        
        // 查找@Priority注解的bean
        String priorityBeanName = determineHighestPriorityCandidate(matchingBeans);
        if (priorityBeanName != null) {
            return matchingBeans.get(priorityBeanName);
        }
        
        // 最后按名称匹配
        if (matchingBeans.containsKey(beanName)) {
            return matchingBeans.get(beanName);
        }
    }
    
    return matchingBeans.values().iterator().next();
}
```

#### 第三步：执行注入

```java
// 字段注入
private class AutowiredFieldElement extends InjectionMetadata.InjectedElement {
    @Override
    protected void inject(Object bean, String beanName, PropertyValues pvs) {
        Field field = (Field) this.member;
        Object value = resolveAutowiredDependency(descriptor, beanName, autowiredBeanNames, typeConverter);
        
        if (value != null) {
            ReflectionUtils.makeAccessible(field);
            field.set(bean, value); // 反射设置字段值
        }
    }
}

// 方法注入
private class AutowiredMethodElement extends InjectionMetadata.InjectedElement {
    @Override
    protected void inject(Object bean, String beanName, PropertyValues pvs) {
        Method method = (Method) this.member;
        Object[] arguments = resolveMethodArguments(method, bean, beanName);
        
        if (arguments != null) {
            ReflectionUtils.makeAccessible(method);
            method.invoke(bean, arguments); // 反射调用方法
        }
    }
}
```

### 4. 解决循环依赖

@Autowired注入支持通过**三级缓存**解决循环依赖：

```java
// Spring的三级缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

## 衍生知识点

### 1. 注入方式的选择建议

- **构造器注入**：推荐使用，保证依赖的不可变性和必需性
- **字段注入**：简洁但难以测试，不推荐在生产环境使用
- **setter注入**：适合可选依赖

### 2. 常见问题及解决方案

#### 问题1：找不到唯一的bean

```java
// 解决方案1：使用@Qualifier
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;

// 解决方案2：使用@Primary
@Primary
@Service
public class UserServiceImpl implements UserService {}

// 解决方案3：使用@Resource指定名称
@Resource(name = "userServiceImpl")
private UserService userService;
```

#### 问题2：循环依赖

```java
// 解决方案1：使用@Lazy延迟初始化
@Autowired
@Lazy
private CircularService circularService;

// 解决方案2：重新设计避免循环依赖
// 解决方案3：使用setter注入替代构造器注入
```

这样的设计使得Spring的依赖注入既灵活又强大，能够处理各种复杂的依赖关系场景。