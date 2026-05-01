# Java为什么要做双亲委派，怎么打破双亲委派，说一个场景

## 为什么要做双亲委派

双亲委派机制的核心目的是**保证类加载的安全性和一致性**：

**1. 避免类的重复加载**

- 如果父类加载器已经加载过某个类，子类加载器就不会再次加载
- 保证JVM中同一个类只被加载一次

**2. 保证核心API的安全性**

- 防止用户自定义的类替换系统核心类
- 比如你无法创建一个自己的`java.lang.String`来替换系统的String类
- 即使你写了一个`java.lang.String`，也会被Bootstrap ClassLoader拦截，加载系统的String

**3. 保证类的唯一性**

- 同一个类在JVM中的唯一性由"类加载器+类的全限定名"决定
- 双亲委派确保相同的类总是由相同的类加载器加载

## 双亲委派的工作流程

```
Application ClassLoader
        ↓ (委派)
Extension ClassLoader  
        ↓ (委派)
Bootstrap ClassLoader
        ↓ (如果找不到，返回给下级)
Extension ClassLoader
        ↓ (如果找不到，返回给下级)
Application ClassLoader (最终加载)
```

## 如何打破双亲委派

**1. 重写ClassLoader的loadClass方法**

```java
public class CustomClassLoader extends ClassLoader {
    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 不调用父类的loadClass，直接自己加载
        if (name.startsWith("com.mycompany")) {
            return findClass(name);
        }
        // 其他类还是走双亲委派
        return super.loadClass(name);
    }
}
```

**2. 使用线程上下文类加载器**

```java
// 获取线程上下文类加载器
ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
// 使用它来加载类
Class<?> clazz = contextClassLoader.loadClass("com.example.MyClass");
```

## 典型应用场景：热部署

**场景描述：** 在开发环境中，我们希望修改代码后不重启应用就能看到效果。

**实现思路：**

```java
public class HotDeployClassLoader extends ClassLoader {
    private String classPath;
    
    public HotDeployClassLoader(String classPath) {
        this.classPath = classPath;
    }
    
    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 对于我们要热部署的类，每次都重新加载
        if (name.startsWith("com.hotdeploy")) {
            return findClass(name);
        }
        // 其他类走正常流程
        return super.loadClass(name);
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 从文件系统读取最新的.class文件
        byte[] classData = loadClassData(name);
        return defineClass(name, classData, 0, classData.length);
    }
    
    private byte[] loadClassData(String className) {
        // 读取.class文件的字节码
        String fileName = classPath + "/" + className.replace('.', '/') + ".class";
        // ... 文件读取逻辑
        return classBytes;
    }
}
```

**使用示例：**

```java
public class HotDeployDemo {
    public static void main(String[] args) throws Exception {
        HotDeployClassLoader loader1 = new HotDeployClassLoader("/path/to/classes");
        Class<?> clazz1 = loader1.loadClass("com.hotdeploy.BusinessService");
        Object instance1 = clazz1.newInstance();
        
        // 修改代码后，创建新的类加载器
        HotDeployClassLoader loader2 = new HotDeployClassLoader("/path/to/classes");
        Class<?> clazz2 = loader2.loadClass("com.hotdeploy.BusinessService");
        Object instance2 = clazz2.newInstance();
        
        // instance1和instance2是不同版本的类的实例
        System.out.println(clazz1 == clazz2); // false，因为类加载器不同
    }
}
```

## 其他打破双亲委派的场景

**1. JDBC驱动加载**

- DriverManager由Bootstrap ClassLoader加载
- 但具体的数据库驱动由Application ClassLoader加载
- 通过Service Provider Interface (SPI)机制实现

**2. Spring框架**

- 使用线程上下文类加载器来加载应用中的Bean
- 实现了灵活的类加载策略

**3. OSGi模块系统**

- 每个Bundle都有自己的类加载器
- 可以实现模块间的类隔离和版本管理

双亲委派机制是JVM类加载的基础安全机制，但在某些特殊场景下，我们需要打破它来实现更灵活的类加载策略。