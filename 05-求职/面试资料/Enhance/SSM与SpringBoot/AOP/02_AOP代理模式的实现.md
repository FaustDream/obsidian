# AOP代理模式的实现？

AOP代理模式主要有两种实现方式：**JDK动态代理**和**CGLIB代理**。让我详细解释这两种实现机制。

## 1. JDK动态代理

**实现原理：**

- 基于Java反射机制
- 只能代理实现了接口的类
- 通过Proxy.newProxyInstance()创建代理对象

**代码示例：**

```java
// 目标接口
public interface UserService {
    void addUser(String name);
}

// 目标实现类
public class UserServiceImpl implements UserService {
    @Override
    public void addUser(String name) {
        System.out.println("添加用户：" + name);
    }
}

// JDK动态代理处理器
public class JdkProxyHandler implements InvocationHandler {
    private Object target;
    
    public JdkProxyHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法执行前 - 日志记录");
        Object result = method.invoke(target, args);
        System.out.println("方法执行后 - 清理工作");
        return result;
    }
    
    // 创建代理对象
    public Object getProxy() {
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            this
        );
    }
}
```

## 2. CGLIB代理

**实现原理：**

- 基于字节码技术，使用ASM框架
- 通过继承目标类生成子类代理
- 可以代理没有接口的类

**代码示例：**

```java
// 目标类（无需实现接口）
public class OrderService {
    public void createOrder(String orderId) {
        System.out.println("创建订单：" + orderId);
    }
}

// CGLIB代理处理器
public class CglibProxyHandler implements MethodInterceptor {
    private Object target;
    
    public CglibProxyHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object intercept(Object obj, Method method, Object[] args, 
                          MethodProxy methodProxy) throws Throwable {
        System.out.println("CGLIB - 方法执行前");
        Object result = methodProxy.invokeSuper(obj, args);
        System.out.println("CGLIB - 方法执行后");
        return result;
    }
    
    // 创建代理对象
    public Object getProxy() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }
}
```

## 3. Spring AOP中的选择策略

Spring框架会根据以下规则自动选择代理方式：

```java
// Spring AOP代理选择逻辑
public class DefaultAopProxyFactory implements AopProxyFactory {
    @Override
    public AopProxy createAopProxy(AdvisedSupport config) {
        if (config.isOptimize() || config.isProxyTargetClass() 
            || hasNoUserSuppliedProxyInterfaces(config)) {
            // 使用CGLIB代理
            return new CglibAopProxy(config);
        } else {
            // 使用JDK动态代理
            return new JdkDynamicAopProxy(config);
        }
    }
}
```

## 4. 两种代理方式的对比

|特性|JDK动态代理|CGLIB代理|
|---|---|---|
|**代理条件**|必须有接口|无需接口|
|**实现方式**|反射机制|字节码生成|
|**性能**|调用时较慢|创建时较慢，调用较快|
|**继承关系**|实现接口|继承目标类|
|**final方法**|可代理|无法代理|

## 5. 实际应用场景

**事务管理示例：**

```java
@Service
@Transactional
public class AccountService {
    public void transfer(String from, String to, BigDecimal amount) {
        // 业务逻辑
        // Spring会自动创建代理，在方法前后添加事务管理
    }
}
```

**日志切面示例：**

```java
@Aspect
@Component
public class LoggingAspect {
    @Around("@annotation(Loggable)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - start;
        System.out.println("方法执行时间：" + executionTime + "ms");
        return result;
    }
}
```

## 6. 扩展知识点

**代理模式的优缺点：**

- **优点：** 增强功能、解耦、动态性
- **缺点：** 性能开销、复杂性增加

**Spring Boot中的配置：**

```properties
# 强制使用CGLIB代理
spring.aop.proxy-target-class=true
```

这种代理机制是Spring框架实现IoC和AOP的核心技术，理解其原理对于深入掌握Spring框架至关重要。