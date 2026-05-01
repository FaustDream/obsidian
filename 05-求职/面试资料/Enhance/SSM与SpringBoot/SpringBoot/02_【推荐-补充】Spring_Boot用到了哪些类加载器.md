# Spring Boot用到了哪些类加载器

Spring Boot中主要使用以下几种类加载器：

## 1. Bootstrap ClassLoader（启动类加载器）

- 负责加载JVM核心类库，如`java.lang.*`、`java.util.*`等
- 由C++实现，在Java中表现为null
- 加载`$JAVA_HOME/jre/lib`目录下的核心jar包

## 2. Extension ClassLoader（扩展类加载器）

- 加载`$JAVA_HOME/jre/lib/ext`目录下的扩展jar包
- 继承自URLClassLoader

## 3. Application ClassLoader（应用类加载器）

- 也称为System ClassLoader
- 加载classpath下的应用程序类
- **这是Spring Boot应用的主要类加载器**

## 4. Spring Boot特有的类加载器

### LaunchedURLClassLoader

- Spring Boot打包成可执行jar后的专用类加载器
- 负责加载jar包内的BOOT-INF/classes和BOOT-INF/lib目录下的类
- 支持嵌套jar的加载机制

### RestartClassLoader（开发时）

- 在使用spring-boot-devtools时启用
- 实现热重启功能
- 将应用类和第三方依赖分别用不同的类加载器加载

## 类加载器的层次结构

```
Bootstrap ClassLoader
       ↓
Extension ClassLoader  
       ↓
Application ClassLoader
       ↓
LaunchedURLClassLoader (生产环境)
或
RestartClassLoader (开发环境)
```

## 相关知识拓展

### 双亲委派模型

Spring Boot遵循Java的双亲委派模型：

- 类加载请求首先委派给父类加载器
- 只有父类加载器无法加载时，子类加载器才会尝试加载
- 保证了Java核心类库的安全性

### Fat Jar的类加载机制

Spring Boot的可执行jar包采用特殊的目录结构：

```
myapp.jar
├── META-INF/
├── BOOT-INF/
│   ├── classes/     # 应用程序类
│   └── lib/         # 依赖的jar包
└── org/springframework/boot/loader/  # Spring Boot加载器
```

LaunchedURLClassLoader通过自定义的URL处理器来读取嵌套jar中的类文件。

### 类加载器隔离的应用场景

- **插件系统**：不同插件使用独立的类加载器，避免版本冲突
- **热部署**：通过替换类加载器实现代码的动态更新
- **多租户应用**：为不同租户使用独立的类加载器实现隔离

这种类加载器设计使得Spring Boot能够将所有依赖打包成单一的可执行jar文件，极大简化了部署过程。