# Java基础——Java代码规范详细版

## 1. 标识符命名规范

### 1.1 概述
标识符命名要统一、达意和简洁。

#### 1.1.1 统一
对于同一概念，应使用一致的命名。例如，供应商可以统一命名为 `supplier` 或 `provider`，避免混用。

#### 1.1.2 达意
标识符应能准确表达其含义。避免使用 `supplier1`、`service2` 等不明确的命名。

#### 1.1.3 简洁
在达意和统一的前提下，保持简洁。例如，避免过长的命名如 `theOrderNameOfTheTargetSupplierWhichIsTransfered`。

#### 1.1.4 骆驼法则
大多数情况下使用骆驼法则（首字母小写，后续单词首字母大写），如：`supplierName`，`addNewContract`。

#### 1.1.5 英文 vs 拼音
尽量使用通俗易懂的英文单词，不要混用拼音和英文。

### 1.2 包名
- 使用小写字母：`com.xxx.settlement`
- 单词间不使用下划线：`com.xxx.settlement.jsfutil`

### 1.3 类名

#### 1.3.1 首字母大写
类名首字母大写，如：`SupplierService`。

#### 1.3.2 后缀
根据不同功能添加后缀，如：
- `Service`：服务类 `PaymentOrderService`
- `Impl`：实现类 `PaymentOrderServiceImpl`
- `Inter`：接口 `LifeCycleInter`
- `Dao`：数据访问类 `PaymentOrderDao`
- `Action`：页面逻辑类 `UpdateOrderListAction`
- `Listener`：事件响应类 `PaymentSuccessListener`

### 1.4 方法名
- 首字母小写，动词在前，如：`addOrder()`。
- 常见动词前缀：
  - `create`：创建 `createOrder()`
  - `delete`：删除 `deleteOrder()`
  - `add`：添加 `addPaidOrder()`
  - `remove`：移除 `removeOrder()`
  - `get`：获取 `getName()`
  - `set`：设置 `setName()`

### 1.5 域（field）名

#### 1.5.1 静态常量
使用全大写，单词间用下划线分隔，如：
```java
public static final String ORDER_PAID_EVENT = "ORDER_PAID_EVENT";
````
#### 1.5.2 枚举

使用全大写，单词间用下划线分隔，如：

```java
public enum Events {
    ORDER_PAID,
    ORDER_CREATED
}
```

#### 1.5.3 其他

使用首字母小写，骆驼法则，如：

```java
public String orderName;
```

### 1.6 局部变量名

参数和局部变量名使用首字母小写，遵循骆驼法则。

## 2. 代码格式

### 2.1 源文件编码

使用 UTF-8 编码，结尾使用 Unix 风格换行（`\n`）。

### 2.2 行宽

行宽不超过 80 字符。

### 2.3 包的导入

删除无用的导入，避免使用通配符导入。可以使用 `ctrl+shift+o` 自动修正。

### 2.4 类格式

* 每个类单独放在一个文件中。
* 类之间加空行。

### 2.5 方法格式

* 每个方法之间加空行。
* 每个方法声明只包含一个返回类型。

### 2.6 代码块格式

#### 2.6.1 缩进风格

大括号开始在同一行，结束大括号与代码块对齐，如：

```java
if (condition) {
    // do something
}
```

#### 2.6.2 空格的使用

* 表达式两边使用空格：

  ```java
  if (a > b) {
      // do something
  }
  ```

* 二元和三元运算符两边加空格：

  ```java
  a + b = c;
  return a == b ? 1 : 0;
  ```

* 逗号后加空格：

  ```java
  call(a, b, c);
  ```

#### 2.6.3 空行的使用

空行用于分隔不同逻辑块，增强代码可读性：

```java
order = orderDao.findOrderById(id);

// update properties
order.setUserName(userName);
order.setPrice(456);
order.setStatus(PAID);
```

## 3. 注释规范

### 3.1 注释 vs 代码

* 注释应简洁且有意义，不应过多或误导。
* 代码清晰时，不需要过多注释。
* 注释应与代码同步。

### 3.2 Java Doc

类、域、方法等的注释使用 JavaDoc 格式：

```java
/**
 * This is a class comment
 */
public class TestClass {
    /**
     * This is a field comment
     */
    public String name;

    /**
     * This is a method comment
     */
    public void call() {
        // do something
    }
}
```

### 3.3 块级别注释

* 单行注释使用 `//`，多行注释使用 `/* */`。
* 长代码块使用标识注释：

  ```java
  /*---------- start: 订单处理 ------- */
  // 取得dao
  OrderDao dao = Factory.getDao("OrderDao");
  /* 查询订单 */
  Order order = dao.findById(456);
  /*---------- end: 订单处理 ------- */
  ```

### 3.4 行内注释

行内注释使用 `//`，放在行尾：

```java
int x = 10; // 初始化变量x
```

## 4. 最佳实践和禁忌

### 4.1 每次保存时优化代码

每次保存代码时，保持代码格式的整洁，避免拖延。

### 4.2 使用日志代替 `System.out.println()`

使用 `log` 提供更灵活的输出，避免使用 `System.out.println()`。

### 4.3 避免省略 `if`, `while`, `for` 的大括号

```java
if (a > b) {
    a++;
}
```

### 4.4 使用 `TODO`

在代码中加入 `//TODO`，方便以后提醒自己完成未做的任务：

```java
// TODO: 完成订单处理逻辑
```

### 4.5 避免多层嵌套

减少代码的嵌套层级，通过合并条件、使用 `return` 或提取方法来简化代码。

### 4.6 减少代码重复

提取重复的代码块，保持代码的单一职责，增强可维护性。

### 4.7 每个方法单一职责

确保每个方法有明确且单一的功能，降低复杂度。

### 4.8 减少变量的作用域

将变量的声明和初始化尽量放在一起，减少变量的作用域。

### 4.9 使用适当的数据结构

根据需求使用合适的数据结构，提高代码效率和可读性。

```

这份 Markdown 格式的 **Java代码规范** 文档已经根据您的要求进行优化，力求简洁明了并包含了必要的代码块。可以直接复制使用。如果有任何进一步的问题，欢迎随时向我提问！
```
