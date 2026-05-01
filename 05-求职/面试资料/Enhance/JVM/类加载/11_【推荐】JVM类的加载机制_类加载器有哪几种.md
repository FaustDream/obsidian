# JVM类的加载机制，类加载器有哪几种

## JVM类加载机制

### 类加载过程

JVM的类加载机制是指虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型的过程。

类加载的完整生命周期包括：

1. **加载（Loading）** - 将class文件字节码内容加载到内存中
2. **验证（Verification）** - 确保Class文件的字节流符合JVM规范
3. **准备（Preparation）** - 为类变量分配内存并设置默认初始值
4. **解析（Resolution）** - 将符号引用替换为直接引用
5. **初始化（Initialization）** - 执行类构造器方法
6. **使用（Using）** - 对象的实际使用
7. **卸载（Unloading）** - 类的卸载

### 类加载器的种类

#### 1. 启动类加载器（Bootstrap ClassLoader）

- **实现**：用C++实现，是JVM的一部分
- **加载路径**：`$JAVA_HOME/lib`目录，或`-Xbootclasspath`参数指定的路径
- **加载内容**：Java核心库，如`java.lang.*`、`java.util.*`等
- **父加载器**：无

#### 2. 扩展类加载器（Extension ClassLoader）

- **实现**：`sun.misc.Launcher$ExtClassLoader`
- **加载路径**：`$JAVA_HOME/lib/ext`目录，或`java.ext.dirs`系统属性指定的路径
- **加载内容**：Java扩展库
- **父加载器**：启动类加载器

#### 3. 应用程序类加载器（Application ClassLoader）

- **实现**：`sun.misc.Launcher$AppClassLoader`
- **加载路径**：classpath路径，即`java.class.path`系统属性指定的路径
- **加载内容**：用户自定义的类和第三方库
- **父加载器**：扩展类加载器

#### 4. 自定义类加载器（Custom ClassLoader）

- **实现**：继承`java.lang.ClassLoader`类
- **使用场景**：热部署、加密类文件、从网络加载类等
- **父加载器**：通常是应用程序类加载器

### 双亲委派模型

**工作原理**：当一个类加载器收到类加载请求时，它首先不会自己去尝试加载这个类，而是把请求委派给父类加载器去完成。只有当父加载器反馈自己无法完成加载请求时，子加载器才会尝试自己去加载。

**优点**：

- 避免类的重复加载
- 保证Java核心API不被篡改
- 提供统一的类加载机制

### 相关扩展知识

#### 打破双亲委派模型的场景

1. **线程上下文类加载器**：解决SPI（Service Provider Interface）问题
2. **热部署**：OSGi框架使用平级类加载器
3. **Web容器**：Tomcat使用WebAppClassLoader优先加载Web应用的类

#### 类加载的时机

- 遇到`new`、`getstatic`、`putstatic`、`invokestatic`指令
- 使用反射调用类
- 初始化子类时发现父类还没初始化
- 虚拟机启动时的主类
- 使用动态语言支持时

#### 常见面试扩展问题

1. **如何自定义类加载器？** 继承ClassLoader并重写findClass方法
2. **什么是类的唯一性？** 同一个类文件被不同类加载器加载，会产生不同的Class对象
3. **如何判断两个类是否相同？** 必须由同一个类加载器加载且类本身相同

这种类加载机制确保了Java程序的安全性和稳定性，是JVM架构中非常重要的组成部分。