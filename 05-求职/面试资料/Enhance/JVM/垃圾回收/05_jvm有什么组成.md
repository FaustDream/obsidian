# jvm有什么组成

JVM（Java Virtual Machine）主要由以下几个核心组件组成：

## 1. 类加载器（Class Loader）

负责将.class文件加载到内存中，包括：

- **Bootstrap ClassLoader**：加载核心Java类库（如rt.jar）
- **Extension ClassLoader**：加载扩展类库
- **Application ClassLoader**：加载应用程序类路径下的类
- **自定义ClassLoader**：用户自定义的类加载器

## 2. 运行时数据区（Runtime Data Area）

这是JVM内存管理的核心部分：

### 线程共享区域：

- **方法区（Method Area）**：存储类信息、常量、静态变量等
- **堆（Heap）**：存储对象实例，分为年轻代和老年代

### 线程私有区域：

- **程序计数器（PC Register）**：记录当前线程执行的字节码指令地址
- **虚拟机栈（VM Stack）**：存储方法调用的栈帧信息
- **本地方法栈（Native Method Stack）**：为本地方法服务

## 3. 执行引擎（Execution Engine）

负责执行字节码：

- **解释器（Interpreter）**：逐行解释执行字节码
- **即时编译器（JIT Compiler）**：将热点代码编译成本地机器码
- **垃圾收集器（Garbage Collector）**：自动回收不再使用的内存

## 4. 本地方法接口（Native Interface）

- **JNI（Java Native Interface）**：允许Java代码调用本地方法
- **本地方法库**：包含JVM需要的本地方法实现

## 相关扩展知识

**内存模型演进**：

- Java 8之前：永久代（PermGen）存储类信息
- Java 8之后：元空间（Metaspace）替代永久代，使用本地内存

**性能优化要点**：

- 堆内存调优：-Xms、-Xmx参数设置
- 垃圾收集器选择：G1、ZGC、Shenandoah等
- JIT编译优化：C1、C2编译器的分层编译

**常见问题排查**：

- OutOfMemoryError：通常与堆、方法区内存不足相关
- StackOverflowError：与虚拟机栈深度超限相关

这种组件化设计使得JVM能够实现"一次编写，到处运行"的跨平台特性，同时提供了高效的内存管理和代码执行能力。