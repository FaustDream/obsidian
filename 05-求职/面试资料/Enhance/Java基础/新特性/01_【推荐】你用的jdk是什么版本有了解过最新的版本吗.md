# 你用的jdk是什么版本？有了解过最新的版本吗？

我来为你详细介绍每个主要JDK版本的核心新特性：

## JDK 8 (2014年) - 里程碑版本

**核心特性：**

- **Lambda表达式** - 函数式编程支持

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println(name));
```

- **Stream API** - 集合的函数式操作

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream().filter(n -> n % 2 == 0).mapToInt(n -> n).sum();
```

- **方法引用** - 简化Lambda表达式
- **接口默认方法** - 接口可以有实现
- **Optional类** - 避免空指针异常
- **新的日期时间API** - LocalDate, LocalTime, LocalDateTime
- **CompletableFuture** - 异步编程支持
- **Nashorn JavaScript引擎**

## JDK 9 (2017年)

**核心特性：**

- **模块系统(Project Jigsaw)** - 将JDK模块化

```java
module com.example.myapp {
    requires java.base;
    exports com.example.api;
}
```

- **JShell** - Java交互式编程环境
- **集合工厂方法**

```java
List<String> list = List.of("a", "b", "c");
Map<String, Integer> map = Map.of("key1", 1, "key2", 2);
```

- **Stream API增强** - takeWhile, dropWhile, ofNullable
- **私有接口方法**
- **多版本JAR包支持**
- **G1成为默认垃圾收集器**

## JDK 10 (2018年)

**核心特性：**

- **局部变量类型推断(var关键字)**

```java
var list = new ArrayList<String>();
var map = Map.of("key", "value");
```

- **应用类数据共享(AppCDS)**
- **垃圾收集器接口**
- **并行全垃圾收集器G1**

## JDK 11 (2018年) - LTS版本

**核心特性：**

- **HTTP Client API** - 标准化HTTP客户端

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .build();
```

- **字符串新方法**

```java
String text = " hello world ";
text.isBlank();     // 检查是否为空白
text.strip();       // 去除前后空格
text.repeat(3);     // 重复字符串
```

- **文件读写简化**

```java
String content = Files.readString(Path.of("file.txt"));
Files.writeString(Path.of("output.txt"), "Hello World");
```

- **Lambda参数var支持**
- **移除JavaFX、Java EE和CORBA模块**
- **Epsilon垃圾收集器**

## JDK 12 (2019年)

**核心特性：**

- **Switch表达式(预览)**

```java
String result = switch (day) {
    case MONDAY, FRIDAY -> "Work day";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};
```

- **字符串新方法** - indent(), transform()
- **Shenandoah垃圾收集器**

## JDK 13 (2019年)

**核心特性：**

- **文本块(预览)** - 多行字符串

```java
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;
```

- **Switch表达式增强**
- **ZGC增强**

## JDK 14 (2020年)

**核心特性：**

- **instanceof模式匹配(预览)**

```java
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

- **Record类(预览)** - 数据载体类

```java
record Person(String name, int age) {}
```

- **文本块标准化**
- **有用的NullPointerException信息**

## JDK 15 (2020年)

**核心特性：**

- **密封类(预览)** - 限制继承

```java
public sealed class Shape permits Circle, Rectangle, Triangle {}
```

- **文本块标准化**
- **隐藏类**
- **ZGC和Shenandoah生产可用**

## JDK 16 (2021年)

**核心特性：**

- **Record类标准化**
- **instanceof模式匹配标准化**
- **Unix域套接字支持**
- **Vector API(孵化)**

## JDK 17 (2021年) - LTS版本

**核心特性：**

- **密封类标准化**

```java
public sealed class Animal permits Dog, Cat {
    // 只有Dog和Cat可以继承Animal
}
```

- **Switch模式匹配(预览)**
- **强封装JDK内部API**
- **移除实验性AOT和JIT编译器**
- **新的macOS渲染管道**

## JDK 18 (2022年)

**核心特性：**

- **UTF-8默认字符集**
- **简单Web服务器**

```java
jwebserver -p 8000 -d /path/to/directory
```

- **代码片段在Javadoc中**
- **Vector API(第二次孵化)**

## JDK 19 (2022年)

**核心特性：**

- **Virtual Threads(预览)** - 轻量级线程

```java
Thread.startVirtualThread(() -> {
    // 虚拟线程任务
});
```

- **结构化并发(孵化)**
- **Pattern Matching for switch(预览)**

## JDK 20 (2023年)

**核心特性：**

- **作用域值(孵化)**
- **Record模式(预览)**
- **Virtual Threads(第二次预览)**
- **结构化并发(第二次孵化)**

## JDK 21 (2023年) - LTS版本

**核心特性：**

- **Virtual Threads标准化** - 大幅提升并发性能

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // 任务代码
    });
}
```

- **Sequenced Collections** - 有序集合接口

```java
SequencedSet<String> set = new LinkedHashSet<>();
set.addFirst("first");
set.addLast("last");
```

- **Pattern Matching for switch标准化**

```java
switch (obj) {
    case Integer i when i > 0 -> "Positive integer";
    case String s -> "String: " + s;
    case null -> "Null value";
    default -> "Other";
}
```

- **String Templates(预览)**
- **结构化并发(预览)**

## JDK 22 (2024年)

**核心特性：**

- **Foreign Function & Memory API标准化**
- **未命名变量和模式**

```java
// 使用_表示不使用的变量
switch (obj) {
    case Point(var x, var _) -> x; // 忽略y坐标
}
```

- **语句before super()**
- **Stream Gatherers(预览)**

## JDK 23 (2024年)

**核心特性：**

- **Primitive types in patterns, instanceof, and switch(预览)**
- **Module import declarations(预览)**
- **Markdown Documentation Comments**
- **ZGC分代收集**

## JDK 24 (2025年)

**核心特性：**

- **Pattern matching改进**
- **结构化并发增强**
- **Vector API持续改进**
- **外部函数接口优化**

## 版本选择建议

**企业项目推荐：**

- **保守型**：JDK 8/11 - 最稳定，生态最完善
- **平衡型**：JDK 17 - 现代特性与稳定性兼顾
- **前沿型**：JDK 21 - 最新LTS，性能和特性最优

**学习发展路径：**

1. 掌握JDK 8的核心特性（Lambda、Stream）
2. 了解JDK 11的实用改进
3. 学习JDK 17的现代化特性
4. 关注JDK 21的并发编程革新

每个版本都在性能、安全性和开发体验上有所提升，选择合适的版本需要平衡项目需求、团队技能和维护成本。