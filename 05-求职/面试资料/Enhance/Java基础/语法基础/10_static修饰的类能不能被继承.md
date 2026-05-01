# static修饰的类能不能被继承

`static` 修饰的类是**可以被继承的**，但有一些重要限制。

## 静态内部类的继承规则

### 1. 基本继承语法

```java
public class OuterClass {
    static class StaticInnerClass {
        void method() {
            System.out.println("静态内部类方法");
        }
    }
}

// 继承静态内部类
class SubClass extends OuterClass.StaticInnerClass {
    @Override
    void method() {
        System.out.println("子类重写的方法");
    }
}
```

### 2. 继承限制说明

**重要限制**：

- 只有**内部类**才能用 `static` 修饰
- **顶级类**（外部类）不能用 `static` 修饰
- 静态内部类继承时必须使用完整的类名路径

### 3. 实际应用示例

```java
public class DatabaseConnection {
    // 静态内部类 - 单例模式的经典应用
    static class ConnectionHolder {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
        
        public static DatabaseConnection getInstance() {
            return INSTANCE;
        }
    }
    
    // 私有构造器
    private DatabaseConnection() {}
}

// 继承静态内部类来扩展功能
class EnhancedConnectionHolder extends DatabaseConnection.ConnectionHolder {
    public static DatabaseConnection getInstanceWithLogging() {
        System.out.println("获取数据库连接实例");
        return getInstance();
    }
}
```

## 相关知识扩展

### 静态内部类的特点

1. **独立性**：不依赖外部类实例，可以直接创建
2. **访问限制**：==只能访问外部类的静态成员==
3. **内存效率**：不持有外部类引用，避免内存泄漏

### 与普通内部类的区别

```java
public class Example {
    class InnerClass {          // 普通内部类
        // 隐式持有外部类引用
    }
    
    static class StaticInnerClass {  // 静态内部类
        // 不持有外部类引用
    }
}
```

### 实际开发建议

- 当内部类不需要访问外部类实例时，优先使用静态内部类
- ==静态内部类常用于工具类、建造者模式、单例模式等场景==
- 继承静态内部类时要注意包访问权限和类的可见性

这样的设计既保持了代码的组织性，又提供了良好的扩展性。