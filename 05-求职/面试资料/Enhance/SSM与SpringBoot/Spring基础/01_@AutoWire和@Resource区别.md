# @AutoWire和@Resource区别

## @Autowired和@Resource的区别

### 1. **来源不同**

- **@Autowired**：Spring框架提供的注解，属于Spring生态
- **@Resource**：Java EE标准注解（JSR-250），属于JDK标准

### 2. **装配方式不同**

- **@Autowired**：默认按照**类型**（byType）进行装配
- **@Resource**：默认按照**名称**（byName）进行装配

### 3. **使用示例对比**

```java
// @Autowired示例
@Component
public class UserService {
    @Autowired
    private UserRepository userRepository; // 按类型匹配
    
    @Autowired
    @Qualifier("userRepositoryImpl") // 需要@Qualifier指定名称
    private UserRepository userRepo;
}

// @Resource示例
@Component
public class UserService {
    @Resource
    private UserRepository userRepository; // 按名称匹配
    
    @Resource(name = "userRepositoryImpl") // 直接指定名称
    private UserRepository userRepo;
}
```

### 4. **属性支持**

- **@Autowired**：
    - `required`：是否必须，默认true
    - 需要配合`@Qualifier`指定名称
- **@Resource**：
    - `name`：指定bean名称
    - `type`：指定bean类型
    - 功能更完整

### 5. **匹配策略**

**@Autowired匹配顺序：**

1. 按类型查找
2. 如果找到多个，再按名称匹配
3. 可用@Qualifier指定

**@Resource匹配顺序：**

1. 如果指定name，按名称查找
2. 如果指定type，按类型查找
3. 如果都没指定，先按名称，再按类型

### 6. **使用场景建议**

```java
// 推荐@Autowired的场景
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService; // 类型唯一时
    
    @Autowired
    @Qualifier("alipayService")
    private PaymentService alipayService; // 多实现时
}

// 推荐@Resource的场景
@Service
public class OrderService {
    @Resource(name = "redisTemplate")
    private RedisTemplate<String, Object> redisTemplate; // 明确指定名称
}
```

### 7. **扩展知识**

**其他依赖注入注解：**

- `@Inject`：JSR-330标准，类似@Autowired
- `@Value`：注入配置值
- `@Lazy`：延迟初始化

**最佳实践：**

- Spring项目推荐使用`@Autowired`
- 需要按名称装配时，`@Resource`更简洁
- 避免在同一项目中混用，保持一致性

这样的对比能帮助你在实际开发中选择合适的注解，也是面试中展现Spring基础扎实的重要知识点。