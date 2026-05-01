# 异常捕获语句包含return时，finally与return的执行顺序

在Java中，当异常捕获语句包含return时，finally块与return语句的执行顺序是一个重要的概念。让我详细解释这个执行顺序：

## 执行顺序规则

**finally块总是在方法真正返回之前执行**，即使在try或catch块中遇到了return语句。

## 具体执行流程

1. **try/catch块中的return语句执行**：计算返回值并暂存
2. **finally块执行**：无论是否有异常都会执行
3. **方法真正返回**：返回之前暂存的值

## 代码示例

```java
public class FinallyReturnExample {
    
    // 示例1：基本执行顺序
    public static int test1() {
        try {
            System.out.println("try块执行");
            return 1;
        } catch (Exception e) {
            System.out.println("catch块执行");
            return 2;
        } finally {
            System.out.println("finally块执行");
        }
    }
    
    // 示例2：finally中修改返回值（基本类型）
    public static int test2() {
        int result = 0;
        try {
            result = 1;
            return result;
        } finally {
            result = 2;  // 这个修改不会影响返回值
            System.out.println("finally中result = " + result);
        }
    }
    
    // 示例3：finally中修改返回值（引用类型）
    public static StringBuilder test3() {
        StringBuilder sb = new StringBuilder("initial");
        try {
            return sb;
        } finally {
            sb.append("-modified");  // 这个修改会影响返回值
        }
    }
    
    // 示例4：finally中包含return语句
    public static int test4() {
        try {
            return 1;
        } finally {
            return 2;  // finally中的return会覆盖try中的return
        }
    }
}
```

## 重要特性分析

### 1. 返回值暂存机制

- 基本类型：值被复制暂存，finally中的修改不影响返回值
- 引用类型：引用被暂存，但对象内容的修改会影响返回值

### 2. finally中的return语句

如果finally块中包含return语句，它会**覆盖**try/catch块中的return语句。

```java
public static int dangerousExample() {
    try {
        return 1;
    } finally {
        return 2;  // 最终返回2，而不是1
    }
}
```

### 3. 异常处理中的执行顺序

```java
public static int exceptionExample() {
    try {
        int result = 10 / 0;  // 抛出异常
        return 1;
    } catch (ArithmeticException e) {
        System.out.println("捕获异常");
        return 2;
    } finally {
        System.out.println("finally执行");
        // 即使catch中有return，finally仍会执行
    }
}
```

## 最佳实践建议

1. **避免在finally中使用return**：这会掩盖原始的返回值或异常
2. **谨慎在finally中修改返回值**：可能导致意外的行为
3. **使用finally进行资源清理**：这是finally的主要用途

## 扩展知识点

### try-with-resources的执行顺序

```java
public static int tryWithResourcesExample() {
    try (FileInputStream fis = new FileInputStream("file.txt")) {
        return 1;
    } catch (IOException e) {
        return 2;
    } finally {
        System.out.println("finally执行");
    }
    // 执行顺序：try → 自动关闭资源 → finally → 返回
}
```

### 嵌套try-finally的执行顺序

```java
public static int nestedExample() {
    try {
        try {
            return 1;
        } finally {
            System.out.println("内层finally");
        }
    } finally {
        System.out.println("外层finally");
    }
    // 执行顺序：内层finally → 外层finally → 返回
}
```

这个机制确保了资源能够被正确释放，即使在方法提前返回的情况下，这是Java异常处理机制的重要设计特性。