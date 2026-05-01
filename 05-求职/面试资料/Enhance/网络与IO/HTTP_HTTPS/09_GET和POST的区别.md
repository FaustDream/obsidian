# GET和POST的区别

GET和POST是HTTP协议中最常用的两种请求方法，它们有以下几个主要区别：

## 1. 语义和用途

- **GET**: 用于获取资源，是幂等操作（多次执行结果相同）
- **POST**: 用于提交数据、创建资源或执行有副作用的操作

## 2. 数据传输方式

- **GET**: 参数通过URL查询字符串传递，如 `http://example.com/api?name=张三&age=25`
- **POST**: 数据放在请求体（Request Body）中传输

## 3. 数据长度限制

- **GET**: 受URL长度限制，通常不超过2048个字符
- **POST**: 理论上无限制，实际受服务器配置影响

## 4. 安全性

- **GET**: 参数暴露在URL中，会被记录在服务器日志、浏览器历史等，相对不安全
- **POST**: 数据在请求体中，相对更安全（但仍需HTTPS加密）

## 5. 缓存特性

- **GET**: 可以被缓存，支持浏览器前进/后退
- **POST**: 通常不被缓存，浏览器会提示是否重新提交

## 6. 编码方式

- **GET**: 只能进行URL编码（application/x-www-form-urlencoded）
- **POST**: 支持多种编码方式，如表单编码、JSON、文件上传等

## 实际开发中的应用

```java
// GET请求示例 - 查询用户信息
@GetMapping("/user")
public User getUser(@RequestParam String id) {
    return userService.findById(id);
}

// POST请求示例 - 创建用户
@PostMapping("/user")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}
```

## 扩展知识

### RESTful API设计原则

- GET: 获取资源
- POST: 创建资源
- PUT: 更新整个资源
- PATCH: 部分更新资源
- DELETE: 删除资源

### 面试常见追问

1. **为什么GET请求不适合传递敏感信息？**
    
    - URL会被记录在访问日志中
    - 浏览器历史记录会保存
    - 可能被代理服务器缓存
2. **POST请求一定比GET安全吗？**
    
    - 不一定，没有HTTPS加密时，POST数据同样可能被截获
    - 真正的安全需要依赖HTTPS等加密传输
3. **GET请求能否有请求体？**
    
    - HTTP规范没有明确禁止，但不推荐
    - 很多服务器和代理会忽略GET请求的请求体

这些区别在实际开发中非常重要，正确选择请求方法不仅影响API的语义清晰度，也关系到应用的安全性和性能。