在Java中，除了使用`new`关键字，还有以下几种创建对象的方法：

## 1. 反射机制

通过`Class.newInstance()`或`Constructor.newInstance()`：

```java
// 使用Class.newInstance()（已废弃）
Class<?> clazz = String.class;
Object obj = clazz.newInstance();

// 使用Constructor.newInstance()（推荐）
Constructor<?> constructor = String.class.getConstructor(String.class);
String str = (String) constructor.newInstance("Hello");
```

## 2. 克隆（Clone）

实现`Cloneable`接口并重写`clone()`方法：

```java
public class Person implements Cloneable {
    private String name;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

// 创建对象
Person person1 = new Person();
Person person2 = (Person) person1.clone();
```

## 3. 序列化和反序列化

通过对象序列化后再反序列化：

```java
// 序列化
ByteArrayOutputStream bos = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(bos);
oos.writeObject(originalObject);

// 反序列化创建新对象
ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
ObjectInputStream ois = new ObjectInputStream(bis);
Object newObject = ois.readObject();
```

## 4. 工厂方法

使用类的静态工厂方法：

```java
// String类的工厂方法
String str = String.valueOf(123);

// Collections的工厂方法
List<String> list = Collections.emptyList();

// 自定义工厂方法
public class PersonFactory {
    public static Person createPerson(String name) {
        return new Person(name);
    }
}
```

## 5. 使用第三方库

如使用Spring框架的依赖注入、CGLib动态代理等：

```java
// Spring容器创建对象
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
MyService service = context.getBean(MyService.class);
```

## 6. Unsafe类（不推荐）

使用`sun.misc.Unsafe`类可以绕过构造函数创建对象：

```java
Unsafe unsafe = Unsafe.getUnsafe();
Object obj = unsafe.allocateInstance(MyClass.class);
```

每种方法都有其特定的使用场景，反射常用于框架开发，克隆用于对象复制，序列化用于持久化和网络传输，工厂方法提供更灵活的对象创建方式。