# 对象作为HashMap的key，要重写什么方法

当对象作为HashMap的key时，**必须重写 `hashCode()` 和 `equals()` 方法**。

## 为什么需要重写这两个方法？

HashMap的工作原理是基于哈希表实现的：

1. **存储时**：HashMap会调用key的`hashCode()`方法计算哈希值，确定存储位置
2. **查找时**：先通过`hashCode()`找到对应的桶，然后用`equals()`方法比较具体的key是否相等

## 重写规则和契约

### hashCode()方法规则：

- **一致性**：同一个对象多次调用hashCode()必须返回相同的值
- **相等对象必须有相同哈希码**：如果两个对象equals()返回true，它们的hashCode()必须相同
- **不相等对象最好有不同哈希码**：虽然不强制，但这样能提高HashMap性能

### equals()方法规则：

- **自反性**：x.equals(x) 必须为true
- **对称性**：x.equals(y) 为true，则y.equals(x)也必须为true
- **传递性**：如果x.equals(y)和y.equals(z)都为true，则x.equals(z)也必须为true
- **一致性**：多次调用结果必须相同

## 实际示例

```java
public class Person {
    private String name;
    private int age;
    
    // 构造器
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Person person = (Person) obj;
        return age == person.age && 
               Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

## 不重写会发生什么？

```java
// 不重写equals和hashCode的情况
Person p1 = new Person("张三", 25);
Person p2 = new Person("张三", 25);

Map<Person, String> map = new HashMap<>();
map.put(p1, "员工1");

// 这里会返回null，因为p1和p2的hashCode不同
System.out.println(map.get(p2)); // 输出：null
```

## 相关知识扩展

### 1. 为什么String可以直接作为HashMap的key？

String类已经重写了equals()和hashCode()方法，确保内容相同的字符串有相同的哈希码。

### 2. 不可变对象作为key的优势

- **线程安全**：不可变对象天然线程安全
- **缓存哈希码**：可以缓存hashCode计算结果，提高性能
- **避免意外修改**：防止key被修改导致HashMap行为异常

### 3. 性能考虑

- **哈希碰撞**：好的hashCode()实现能减少碰撞，提高性能
- **计算复杂度**：hashCode()计算不应太复杂，避免影响性能

### 4. 其他Map实现的要求

- **TreeMap**：key需要实现Comparable接口或提供Comparator
- **LinkedHashMap**：同HashMap，需要重写equals()和hashCode()

这样重写后，相同内容的对象就能正确地作为HashMap的key使用了。