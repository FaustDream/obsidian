# Java的修饰符作用域，protect，不加修饰符默认的是

Java中的访问修饰符作用域从大到小排列如下：

## 四种访问修饰符的作用域

> 不同包子类：A1 包中定义了类 A，B1 包中定义了继承于类 A 的子类 B

### 1. **public（公共的）**

- **作用域**：任何地方都可以访问
- **适用范围**：同一个类、同一个包、不同包的子类、不同包的非子类

### 2. **protected（受保护的）**

- **作用域**：同一个包内的所有类 + 不同包中的子类
- **适用范围**：同一个类、同一个包、不同包的子类
- **注意**：不同包的非子类无法访问

### 3. **default（默认的，不加修饰符）**

- **作用域**：仅限同一个包内
- **适用范围**：同一个类、同一个包
- **特点**：也叫包访问权限（package-private）

### 4. **private（私有的）**

- **作用域**：仅限同一个类内
- **适用范围**：只有同一个类

## 代码示例

```java
// 包 com.example.package1
public class Parent {
    public int publicField = 1;        // 任何地方可访问
    protected int protectedField = 2;  // 同包 + 子类可访问
    int defaultField = 3;              // 仅同包可访问
    private int privateField = 4;      // 仅本类可访问
}

// 包 com.example.package2
import com.example.package1.Parent;

public class Child extends Parent {
    public void test() {
        System.out.println(publicField);    // ✅ 可以访问
        System.out.println(protectedField); // ✅ 可以访问（子类）
        // System.out.println(defaultField); // ❌ 编译错误（不同包）
        // System.out.println(privateField); // ❌ 编译错误（私有）
    }
}

// 包 com.example.package2
public class Other {
    public void test() {
        Parent p = new Parent();
        System.out.println(p.publicField);    // ✅ 可以访问
        // System.out.println(p.protectedField); // ❌ 编译错误（非子类）
        // System.out.println(p.defaultField);   // ❌ 编译错误（不同包）
        // System.out.println(p.privateField);   // ❌ 编译错误（私有）
    }
}
```

## 权限表格总结

|修饰符|同一类|同一包|不同包子类|不同包非子类|
|---|---|---|---|---|
|public|✅|✅|✅|✅|
|protected|✅|✅|✅|❌|
|default|✅|✅|❌|❌|
|private|✅|❌|❌|❌|

## 扩展知识点

**1. 类的访问修饰符**

- 外部类只能使用 `public` 或 `default`
- 内部类可以使用所有四种修饰符

**2. 接口中的访问修饰符**

- 接口中的方法默认是 `public abstract`
- 接口中的变量默认是 `public static final`

**3. 继承中的访问权限**

- 子类重写方法时，访问权限只能扩大，不能缩小
- 例如：父类的 `protected` 方法，子类可以改为 `public`，但不能改为 `private`

这种设计体现了Java的封装特性，通过控制访问权限来保护数据安全和维护代码的可维护性。