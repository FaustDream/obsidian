# springboot的核心配置文件

Spring Boot的核心配置文件主要有以下几种：

## 1. application.properties / application.yml

这是Spring Boot最核心的配置文件，用于配置应用程序的各种属性。

**application.properties示例：**

```properties
# 服务器配置
server.port=8080
server.servlet.context-path=/api

# 数据库配置
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

**application.yml示例：**

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

## 2. bootstrap.properties / bootstrap.yml

这是Spring Cloud应用中的配置文件，优先级高于application配置文件，主要用于：

- 配置应用名称
- 配置配置中心相关信息
- 配置服务注册中心信息

```yaml
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev
```

## 3. 环境特定配置文件

Spring Boot支持多环境配置，通过profile来区分：

- `application-dev.properties` - 开发环境
- `application-test.properties` - 测试环境
- `application-prod.properties` - 生产环境

激活方式：

```properties
spring.profiles.active=dev
```

## 4. 配置文件加载顺序

Spring Boot按以下优先级加载配置（数字越小优先级越高）：

1. 命令行参数
2. 系统属性
3. 操作系统环境变量
4. jar包外的application-{profile}.properties
5. jar包内的application-{profile}.properties
6. jar包外的application.properties
7. jar包内的application.properties

## 5. 配置文件位置

Spring Boot会按以下顺序查找配置文件：

1. 当前目录下的/config子目录
2. 当前目录
3. classpath下的/config包
4. classpath根目录

## 6. 常用配置属性分类

**服务器配置：**

```properties
server.port=8080
server.servlet.context-path=/
server.tomcat.max-threads=200
```

**数据源配置：**

```properties
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
spring.datasource.driver-class-name=
```

**JPA/Hibernate配置：**

```properties
spring.jpa.hibernate.ddl-auto=
spring.jpa.show-sql=
spring.jpa.database-platform=
```

**Redis配置：**

```properties
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
```

**日志配置：**

```properties
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.file.name=app.log
```

## 扩展知识点

**1. 配置属性绑定** 可以通过`@ConfigurationProperties`注解将配置文件中的属性绑定到Java对象：

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private String name;
    private String version;
    // getters and setters
}
```

**2. 配置文件加密** 可以使用Jasypt等工具对敏感配置进行加密：

```properties
spring.datasource.password=ENC(加密后的密码)
```

**3. 外部配置** Spring Boot支持从多种外部源加载配置，如配置中心、环境变量、命令行参数等，这为应用的部署和运维提供了很大的灵活性。

这些配置文件是Spring Boot应用的基础，掌握它们的使用方法对于Spring Boot开发至关重要。