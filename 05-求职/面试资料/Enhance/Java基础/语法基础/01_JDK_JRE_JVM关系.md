# JDK、JRE、JVM关系

JDK、JRE、JVM是Java技术栈中三个重要的概念，它们之间存在包含关系：

## JVM (Java Virtual Machine) - Java虚拟机

JVM是Java程序运行的核心环境，负责：

- **字节码执行**：将编译后的.class文件转换为机器码执行
- **内存管理**：堆内存、栈内存、方法区等的分配和回收
- **垃圾回收**：自动管理内存，回收不再使用的对象
- **平台无关性**：实现"一次编译，到处运行"的特性

## JRE (Java Runtime Environment) - Java运行时环境

JRE包含了运行Java程序所需的所有组件：

- **JVM**：核心虚拟机
- **Java核心类库**：如java.lang、java.util等标准API
- **支持文件**：配置文件、属性文件等

JRE是运行Java应用程序的最小环境，但不能用于开发。

## JDK (Java Development Kit) - Java开发工具包

JDK是完整的Java开发环境，包含：

- **JRE**：运行环境
- **开发工具**：
    - javac：Java编译器
    - java：Java解释器
    - javadoc：文档生成工具
    - jar：打包工具
    - jdb：调试器

## 关系总结

```
JDK = JRE + 开发工具(javac、java、jar、javadoc、jps)
JRE = JVM + Java类库
```

**实际应用场景**：

- **开发阶段**：需要安装JDK，用于编写、编译、调试Java程序
- **生产部署**：服务器只需安装JRE即可运行Java应用
- **最终用户**：普通用户运行Java应用只需JRE

## 扩展知识

**JVM的不同实现**：

- Oracle HotSpot JVM（最常用）
- OpenJDK
- IBM J9
- GraalVM

**版本对应关系**： 每个JDK版本都包含对应的JRE和JVM，比如JDK 8包含JRE 8和JVM 8，它们在功能特性上保持一致。

这种分层设计使得Java具有良好的模块化特性，满足不同角色的需求：开发者使用JDK，运维人员部署JRE，而JVM则为所有Java程序提供统一的运行环境。