# ClassLoader有什么用

ClassLoader是Java虚拟机的核心组件之一，主要负责将字节码文件(.class文件)加载到JVM内存中并转换为可执行的Class对象。

## 主要作用

**1. 类加载功能**

- 根据类的全限定名查找并读取.class文件
- 将字节码转换为JVM内部的Class对象
- 为类分配内存空间并初始化

**2. 命名空间隔离**

- 不同ClassLoader加载的同名类被视为不同的类
- 实现了类的隔离机制，避免类冲突

**3. 安全控制**

- 通过双亲委派机制确保核心类库的安全性
- 防止恶意代码替换系统核心类

## 类加载器层次结构

Java中有三种主要的ClassLoader：

**Bootstrap ClassLoader（引导类加载器）**

```java
// 加载java.lang.*等核心类库
// 由C++实现，Java中表示为null
```

**Extension ClassLoader（扩展类加载器）**

```java
// 加载jre/lib/ext目录下的类
// 对应sun.misc.Launcher$ExtClassLoader
```

**Application ClassLoader（应用程序类加载器）**

```java
// 加载classpath下的用户类
// 对应sun.misc.Launcher$AppClassLoader
```

## 双亲委派机制

```java
protected Class<?> loadClass(String name, boolean resolve) {
    // 1. 检查类是否已经加载
    Class<?> c = findLoadedClass(name);
    if (c == null) {
        try {
            // 2. 委派给父加载器
            if (parent != null) {
                c = parent.loadClass(name, false);
            } else {
                c = findBootstrapClassOrNull(name);
            }
        } catch (ClassNotFoundException e) {
            // 3. 父加载器无法加载时，自己尝试加载
            c = findClass(name);
        }
    }
    return c;
}
```

## 自定义ClassLoader示例

```java
public class CustomClassLoader extends ClassLoader {
    private String classPath;
    
    public CustomClassLoader(String classPath) {
        this.classPath = classPath;
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            // 读取字节码文件
            byte[] classData = loadClassData(name);
            // 定义类
            return defineClass(name, classData, 0, classData.length);
        } catch (IOException e) {
            throw new ClassNotFoundException(name);
        }
    }
    
    private byte[] loadClassData(String name) throws IOException {
        String fileName = classPath + File.separator + 
                         name.replace('.', File.separatorChar) + ".class";
        return Files.readAllBytes(Paths.get(fileName));
    }
}
```

## 实际应用场景

**1. 热部署**

- Web容器通过自定义ClassLoader实现应用的热部署
- 新版本类用新的ClassLoader加载，旧版本自动卸载

**2. 插件系统**

- 为每个插件创建独立的ClassLoader
- 实现插件间的类隔离

**3. 代码加密**

- 自定义ClassLoader可以从加密的字节码文件中加载类
- 提供一定程度的代码保护

**4. 动态编译**

- 结合JavaCompiler在运行时编译和加载Java代码
- 支持动态语言特性

## 相关知识扩展

**Class.forName()与ClassLoader.loadClass()区别**

```java
// Class.forName()会触发类的初始化
Class<?> clazz1 = Class.forName("com.example.MyClass");

// ClassLoader.loadClass()只加载不初始化
Class<?> clazz2 = ClassLoader.getSystemClassLoader()
                            .loadClass("com.example.MyClass");
```

**内存泄漏风险**

- ClassLoader持有加载类的引用
- 如果ClassLoader不能被回收，会导致内存泄漏
- 需要及时清理ClassLoader引用

ClassLoader是Java动态性的基础，深入理解其工作原理对于开发高级Java应用非常重要。