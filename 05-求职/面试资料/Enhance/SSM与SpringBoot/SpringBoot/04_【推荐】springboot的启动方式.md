# springboot的启动方式

Spring Boot有多种启动方式，我来为你详细介绍：

## 1. 主类启动方式（最常用）

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这是最标准的启动方式，通过`@SpringBootApplication`注解和`SpringApplication.run()`方法启动应用。

## 2. 自定义SpringApplication启动

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.setWebApplicationType(WebApplicationType.SERVLET);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}
```

通过创建SpringApplication实例，可以在启动前进行各种自定义配置。

## 3. SpringApplicationBuilder方式

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
            .web(WebApplicationType.SERVLET)
            .bannerMode(Banner.Mode.OFF)
            .run(args);
    }
}
```

提供了更加流畅的API来配置SpringApplication。

## 4. 外部容器启动（War包部署）

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

继承`SpringBootServletInitializer`，支持部署到外部Servlet容器如Tomcat。

## 5. 命令行启动方式

### Maven方式

```bash
mvn spring-boot:run
```

### Gradle方式

```bash
./gradlew bootRun
```

### Java命令启动

```bash
java -jar application.jar
```

## 6. IDE启动方式

在IDE中直接运行主类的main方法，这是开发阶段最常用的方式。

## 扩展知识点

### 启动过程中的关键步骤：

1. **创建SpringApplication实例**：解析主配置类，设置应用类型
2. **准备Environment**：加载配置文件，处理命令行参数
3. **创建ApplicationContext**：根据应用类型创建相应的上下文
4. **刷新上下文**：加载Bean定义，实例化Bean，启动内嵌服务器
5. **触发started事件**：通知应用启动完成

### 应用类型区分：

- **SERVLET**：传统的Web应用
- **REACTIVE**：响应式Web应用（WebFlux）
- **NONE**：非Web应用

### 启动监听器：

```java
@Component
public class ApplicationStartupListener implements ApplicationListener<ApplicationReadyEvent> {
    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        System.out.println("应用启动完成");
    }
}
```

### 性能优化启动参数：

```bash
java -Xms512m -Xmx1024m -XX:+UseG1GC -jar application.jar
```

这些启动方式各有适用场景，开发时通常使用IDE或Maven插件，生产环境多使用java -jar命令或容器部署。