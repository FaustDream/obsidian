# autowired和resource区别

## @Autowired 和 @Resource 的区别

### 1. **来源不同**

- **@Autowired**：Spring框架原生注解，属于`org.springframework.beans.factory.annotation`包
- **@Resource**：Java EE标准注解（JSR-250），属于`javax.annotation`包

### 2. **注入方式不同**

- **@Autowired**：默认按**类型**（byType）注入
- **@Resource**：默认按**名称**（byName）注入，找不到同名bean时才按类型注入

### 3. **属性支持不同**

```java
// @Autowired 主要属性
@Autowired(required = false)  // 是否必须注入，默认true
private UserService userService;

// @Resource 主要属性
@Resource(name = "userServiceImpl")  // 指定bean名称
@Resource(type = UserService.class)  // 指定bean类型
private UserService userService;
```

### 4. **使用位置不同**

- **@Autowired**：可用于构造器、方法、参数、字段、注解
- **@Resource**：只能用于字段和setter方法

### 5. **实际使用示例**

```java
@Service
public class UserController {
    
    // @Autowired 按类型注入
    @Autowired
    private UserService userService;
    
    // @Resource 按名称注入
    @Resource(name = "userServiceImpl")
    private UserService userService2;
    
    // 多个实现类的情况
    @Autowired
    @Qualifier("userServiceImpl")  // 配合@Qualifier指定具体实现
    private UserService userService3;
}
```

### 6. **多实现类处理**

当接口有多个实现类时：

- **@Autowired**：需要配合`@Qualifier`注解或`@Primary`注解
- **@Resource**：直接通过name属性指定

### 7. **性能和推荐**

- **性能**：@Resource略优于@Autowired（先按name查找更快）
- **推荐**：Spring项目中推荐使用@Autowired，更符合Spring生态

### 8. **衍生知识点**

**依赖注入的三种方式：**

1. **构造器注入**（推荐）：保证依赖不可变，避免循环依赖
2. **Setter注入**：允许可选依赖，但可能导致对象状态不一致
3. **字段注入**：代码简洁但不利于单元测试

**Spring容器中bean的获取顺序：**

1. 先按名称查找
2. 再按类型查找
3. 最后按限定符查找

这种设计体现了Spring框架的灵活性和Java EE标准的兼容性。