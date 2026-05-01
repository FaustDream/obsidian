# SpringBoot的启动方式有几种

SpringBoot有多种启动方式，我来详细介绍一下：

## 1. 主类启动（最常见）

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这是最标准的启动方式，直接运行main方法即可。

## 2. SpringApplication构建器启动

```java
public static void main(String[] args) {
    new SpringApplicationBuilder(Application.class)
        .web(WebApplicationType.SERVLET)
        .run(args);
}
```

这种方式提供了更多的配置选项，比如可以设置应用类型、配置文件等。

## 3. 可执行JAR包启动

```bash
java -jar application.jar
```

通过`mvn package`或`gradle build`打包后，可以直接运行jar包。

## 4. Maven插件启动

```bash
mvn spring-boot:run
```

开发阶段常用，无需打包直接运行。

## 5. Gradle插件启动

```bash
./gradlew bootRun
```

Gradle项目的启动方式。

## 6. 外部Tomcat启动（传统方式）

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

需要继承`SpringBootServletInitializer`，打包成war包部署到外部Tomcat。

## 7. Docker容器启动

```dockerfile
FROM openjdk:8-jre-slim
COPY application.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

## 8. 系统服务启动

在Linux系统中可以配置为systemd服务：

```bash
sudo systemctl start myapp
```

## 相关扩展知识

**启动流程分析：**

1. 创建SpringApplication实例
2. 准备Environment环境
3. 创建ApplicationContext上下文
4. 加载配置和Bean定义
5. 刷新上下文
6. 启动内嵌服务器（如Tomcat）

**常用启动参数：**

- `--server.port=8080` 指定端口
- `--spring.profiles.active=dev` 激活配置文件
- `--spring.config.location=classpath:/config/` 指定配置文件位置

**内嵌服务器：** SpringBoot默认使用Tomcat，也支持Jetty、Undertow等，可以通过依赖切换：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

这些启动方式在不同的开发和部署场景中都有其适用性，选择合适的方式能够提高开发效率和部署灵活性。