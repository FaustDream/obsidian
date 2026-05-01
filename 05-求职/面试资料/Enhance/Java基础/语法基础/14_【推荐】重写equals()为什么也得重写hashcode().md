# 重写equals()为什么也得重写hashcode()

这是一个非常经典的Java面试问题，涉及到对象相等性判断的核心原理。让我详细解释一下为什么重写equals()必须同时重写hashCode()。

## 核心原因：维护hashCode()与equals()的契约

Java规范明确要求：**如果两个对象通过equals()方法比较相等，那么它们的hashCode()必须返回相同的值**。

### 具体原因分析

**1. HashMap/HashSet等集合类的正确性**

```java
// 错误示例：只重写equals()，不重写hashCode()
public class Person {
    private String name;
    private int age;
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    // 没有重写hashCode()，使用Object的默认实现
}

// 问题演示
Person p1 = new Person("张三", 25);
Person p2 = new Person("张三", 25);

System.out.println(p1.equals(p2)); // true
System.out.println(p1.hashCode() == p2.hashCode()); // false！违反契约

HashSet<Person> set = new HashSet<>();
set.add(p1);
System.out.println(set.contains(p2)); // false！逻辑错误
```

**2. 正确的实现方式**

```java
public class Person {
    private String name;
    private int age;
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

### 深入理解：Hash集合的工作原理

HashMap/HashSet的查找过程：

1. **计算hashCode()** → 确定bucket位置
2. **使用equals()** → 在bucket中精确匹配

如果hashCode()不一致，第1步就会失败，根本不会执行第2步的equals()比较。

### 扩展知识点

**1. hashCode()的设计原则**

- 相等对象必须有相同的hashCode
- 不相等对象最好有不同的hashCode（减少冲突）
- hashCode值在对象生命周期内保持不变

**2. 常见的hashCode()实现策略**

```java
// 使用Objects.hash()（推荐）
@Override
public int hashCode() {
    return Objects.hash(name, age, email);
}

// 手动实现
@Override
public int hashCode() {
    int result = name != null ? name.hashCode() : 0;
    result = 31 * result + age;
    result = 31 * result + (email != null ? email.hashCode() : 0);
    return result;
}
```

**3. 性能考虑**

- 好的hashCode()分布均匀，减少冲突
- 计算不能太复杂，影响性能
- 31是常用的质数，有利于散列分布

**4. 特殊情况处理**

```java
// 不可变对象可以缓存hashCode
private transient int hashCode; // 缓存

@Override
public int hashCode() {
    if (hashCode == 0) {
        hashCode = Objects.hash(name, age);
    }
    return hashCode;
}
```

### 总结

重写equals()必须同时重写hashCode()的根本原因是维护Java对象相等性的基本契约，确保基于哈希的集合类能够正确工作。这不仅是语言规范的要求，更是保证程序逻辑正确性的必要条件。

-----
-----
-----


你提到了一个很好的细节！让我深入解释一下为什么在hashCode()计算中选择31这个质数。

## 为什么选择31？

### 1. 质数的散列优势

**质数的数学特性：**

```java
// 质数31的优势演示
public class HashCodeDemo {
    public static void main(String[] args) {
        // 使用31作为乘数
        System.out.println("abc".hashCode()); // 96354
        System.out.println("abd".hashCode()); // 96355
        System.out.println("ac".hashCode());  // 3105
        
        // 如果使用偶数（如32），更容易产生冲突
        System.out.println(hashWithMultiplier("abc", 32)); // 演示对比
        System.out.println(hashWithMultiplier("abd", 32));
    }
    
    public static int hashWithMultiplier(String s, int multiplier) {
        int hash = 0;
        for (char c : s.toCharArray()) {
            hash = hash * multiplier + c;
        }
        return hash;
    }
}
```

### 2. 31的特殊性质

**位运算优化：**

```java
// 31 * i 可以优化为 (i << 5) - i
// 这是编译器自动优化，性能更好

public int hashCode() {
    int result = name != null ? name.hashCode() : 0;
    // result = 31 * result + age;
    // 实际编译后可能优化为：
    // result = (result << 5) - result + age;
    return result;
}
```

### 3. 经验验证

**Java String类的实现：**

```java
// String.hashCode()的实际实现
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];  // 使用31
        }
        hash = h;
    }
    return h;
}
```

### 4. 为什么不选择其他质数？

**不同质数的对比：**

```java
public class PrimeComparison {
    public static void testDistribution(int prime) {
        Set<Integer> hashes = new HashSet<>();
        // 测试大量字符串的散列分布
        for (int i = 0; i < 10000; i++) {
            String test = "test" + i;
            hashes.add(calculateHash(test, prime));
        }
        System.out.println("Prime " + prime + " unique hashes: " + hashes.size());
    }
    
    private static int calculateHash(String s, int prime) {
        int hash = 0;
        for (char c : s.toCharArray()) {
            hash = hash * prime + c;
        }
        return hash;
    }
}
```

### 5. 实际应用中的考虑

**选择合适的乘数：**

```java
public class OptimizedHashCode {
    private String name;
    private int age;
    private String email;
    
    @Override
    public int hashCode() {
        // 方法1：使用31（经典方案）
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        result = 31 * result + (email != null ? email.hashCode() : 0);
        return result;
    }
    
    // 方法2：使用Objects.hash()（内部也使用31）
    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }
}
```

### 6. 深入分析：散列分布测试

```java
public class HashDistributionTest {
    public static void main(String[] args) {
        int[] multipliers = {31, 37, 41, 43, 47}; // 不同的质数
        
        for (int mult : multipliers) {
            testHashDistribution(mult);
        }
    }
    
    private static void testHashDistribution(int multiplier) {
        Map<Integer, Integer> buckets = new HashMap<>();
        int bucketCount = 1000;
        
        // 生成10000个测试字符串
        for (int i = 0; i < 10000; i++) {
            String test = "user" + i + "@example.com";
            int hash = calculateHash(test, multiplier);
            int bucket = Math.abs(hash) % bucketCount;
            buckets.put(bucket, buckets.getOrDefault(bucket, 0) + 1);
        }
        
        // 分析分布均匀性
        double variance = calculateVariance(buckets.values());
        System.out.println("Multiplier " + multiplier + " variance: " + variance);
    }
    
    private static double calculateVariance(Collection<Integer> values) {
        double mean = values.stream().mapToInt(Integer::intValue).average().orElse(0);
        return values.stream()
                    .mapToDouble(v -> Math.pow(v - mean, 2))
                    .average()
                    .orElse(0);
    }
}
```

### 7. 总结

**31被选择的原因：**

1. **质数特性**：减少哈希冲突，分布更均匀
2. **性能优化**：31 * i = (i << 5) - i，位运算高效
3. **经验验证**：在大量实际应用中表现良好
4. **标准约定**：Java标准库广泛使用，形成惯例

**实际建议：**

- 对于自定义类，直接使用`Objects.hash()`
- 如果手动实现，使用31作为乘数
- 关注散列分布的均匀性比具体数值更重要

31不是唯一选择，但它是经过时间验证的最佳实践之一。