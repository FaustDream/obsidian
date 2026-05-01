# Java异常处理机制，关键字

Java异常处理机制是程序错误管理的核心机制，让我为您详细解析：

## 异常处理的关键字

### 1. try-catch-finally

**try块**：包含可能抛出异常的代码 **catch块**：捕获并处理特定类型的异常 **finally块**：无论是否发生异常都会执行的代码

```java
try {
    // 可能抛出异常的代码
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // 处理算术异常
    System.out.println("除零异常：" + e.getMessage());
} finally {
    // 清理资源，总是执行
    System.out.println("清理资源");
}
```

### 2. throw

用于主动抛出异常对象

```java
public void validateAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年龄不能为负数");
    }
}
```

### 3. throws

在方法声明中声明可能抛出的异常

```java
public void readFile(String fileName) throws IOException {
    FileReader file = new FileReader(fileName);
    // 文件操作
}
```

## 异常的分类体系

### 1. Throwable层次结构

```
Throwable
├── Error (系统级错误，不建议捕获)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── RuntimeException (运行时异常，非受检异常)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── IOException (受检异常，编译时必须处理)
        ├── FileNotFoundException
        └── SQLException
```

### 2. 受检异常 vs 非受检异常

**受检异常(Checked Exception)**：

- 编译时必须处理
- 继承自Exception但不继承RuntimeException
- 例如：IOException, SQLException

**非受检异常(Unchecked Exception)**：

- 运行时异常，编译时不强制处理
- 继承自RuntimeException
- 例如：NullPointerException, ArrayIndexOutOfBoundsException

## 异常处理的最佳实践

### 1. 具体化异常处理

```java
try {
    // 业务代码
} catch (FileNotFoundException e) {
    // 处理文件未找到
    log.error("文件未找到：{}", e.getMessage());
} catch (IOException e) {
    // 处理其他IO异常
    log.error("IO异常：{}", e.getMessage());
} catch (Exception e) {
    // 处理其他异常
    log.error("未知异常：{}", e.getMessage());
}
```

### 2. 自定义异常

```java
public class BusinessException extends Exception {
    private String errorCode;
    
    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}
```

### 3. try-with-resources (Java 7+)

自动资源管理，实现AutoCloseable接口的资源会自动关闭

```java
try (FileReader reader = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(reader)) {
    return br.readLine();
} catch (IOException e) {
    log.error("文件读取异常", e);
    throw new BusinessException("FILE_READ_ERROR", "文件读取失败");
}
```

## 扩展知识点

### 1. 异常链和异常传播

```java
public void methodA() throws BusinessException {
    try {
        methodB();
    } catch (SQLException e) {
        // 异常转换，保留原始异常信息
        throw new BusinessException("DB_ERROR", "数据库操作失败", e);
    }
}
```

### 2. 异常处理的性能考虑

- 异常处理有性能开销，不应用于正常的业务流程控制
- 避免在循环中频繁抛出异常
- 使用异常进行流程控制是反模式

### 3. 多异常处理 (Java 7+)

```java
try {
    // 业务代码
} catch (IOException | SQLException e) {
    // 同时处理多种异常类型
    log.error("操作失败", e);
}
```

### 4. 异常处理的设计原则

- **快速失败**：尽早发现并抛出异常
- **异常安全**：保证程序状态的一致性
- **异常转换**：将底层异常转换为业务层异常
- **日志记录**：记录异常信息便于排查问题

这套异常处理机制确保了Java程序的健壮性和可维护性，是编写高质量Java代码的基础。