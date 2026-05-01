# ioc注解 @resource @autowire @Qualifier

I'll explain these three important IoC (Inversion of Control) annotations in Spring Framework and their differences:

## @Resource

**Origin**: This is a **JSR-250** standard annotation, not specific to Spring.

**Features**:

- **By name first**: Matches beans by name first, then by type
- **Field/setter injection**: Can be used on fields or setter methods
- **Name specification**: Can specify bean name with `@Resource(name="beanName")`

**Example**:

```java
@Component
public class UserService {
    @Resource(name = "userDao")
    private UserDao userDao;
    
    @Resource  // Will look for bean named "emailService"
    private EmailService emailService;
}
```

## @Autowired

**Origin**: This is a **Spring-specific** annotation.

**Features**:

- **By type first**: Matches beans by type first
- **Multiple locations**: Can be used on fields, constructors, methods, and parameters
- **Required control**: Has `required` attribute (default true)
- **Collection injection**: Can inject collections of beans

**Example**:

```java
@Component
public class UserService {
    @Autowired
    private UserDao userDao;
    
    @Autowired(required = false)
    private Optional<CacheService> cacheService;
    
    @Autowired
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

## @Qualifier

**Purpose**: Used with `@Autowired` to specify which bean to inject when multiple candidates exist.

**Features**:

- **Bean disambiguation**: Resolves ambiguity when multiple beans of same type exist
- **Works with @Autowired**: Must be used together with `@Autowired`
- **Custom qualifiers**: Can create custom qualifier annotations

**Example**:

```java
@Component
public class NotificationService {
    @Autowired
    @Qualifier("emailSender")
    private MessageSender emailSender;
    
    @Autowired
    @Qualifier("smsSender")
    private MessageSender smsSender;
}

@Component("emailSender")
public class EmailSender implements MessageSender { }

@Component("smsSender") 
public class SmsSender implements MessageSender { }
```

## Key Differences Summary

|Aspect|@Resource|@Autowired|@Qualifier|
|---|---|---|---|
|**Standard**|JSR-250|Spring|Spring|
|**Matching Strategy**|Name → Type|Type → Name|Works with @Autowired|
|**Usage**|Field, Setter|Field, Constructor, Method, Parameter|With @Autowired only|
|**Required**|Always required|Configurable|N/A|
|**Multiple beans**|Use name attribute|Needs @Qualifier|Solves this problem|

## Advanced Usage Scenarios

### 1. Constructor Injection (Recommended)

```java
@Component
public class UserService {
    private final UserDao userDao;
    private final EmailService emailService;
    
    @Autowired  // Optional in single constructor
    public UserService(UserDao userDao, 
                      @Qualifier("gmailService") EmailService emailService) {
        this.userDao = userDao;
        this.emailService = emailService;
    }
}
```

### 2. Collection Injection

```java
@Component
public class NotificationService {
    @Autowired
    private List<MessageSender> allSenders;  // Injects all MessageSender beans
    
    @Autowired
    private Map<String, MessageSender> senderMap;  // Key = bean name
}
```

### 3. Custom Qualifiers

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.PARAMETER})
public @interface DatabaseType {
    String value();
}

@Component
@DatabaseType("mysql")
public class MySQLUserDao implements UserDao { }

@Component
public class UserService {
    @Autowired
    @DatabaseType("mysql")
    private UserDao userDao;
}
```

## Best Practices

1. **Prefer constructor injection** for mandatory dependencies
2. **Use @Autowired** for Spring applications (better IDE support)
3. **Use @Resource** when working with JEE containers or need name-based injection
4. **Combine @Autowired + @Qualifier** for disambiguation
5. **Consider using @Primary** annotation on beans to avoid @Qualifier everywhere

These annotations are fundamental to Spring's dependency injection mechanism and understanding their differences is crucial for effective Spring development.