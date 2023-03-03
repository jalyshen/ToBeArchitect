## springboot之SpringBootServletInitializer

原文：[springboot之SpringBootServletInitializer - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1749644)



### 1. 概述

对于SpringBoot应用，一般都会打包成jar包，使用内置的容器运行。但是有时候需要使用传统的方式部署，把Springboot应用打包成WAR包，然后部署在外部的容器中运行。使用Jar包运行通常会有Main类启动，但是使用外部容器的话，外部容器无法识别应用启动类，需要在应用中继承 SpringBootServletInitializer 类，然后重写 config 方法，将其指向应用启动类。

这里介绍 SpringBootServletInitializer 的原理和使用。它是 WebApplicationInitializer 的扩展，从部署在 Web 容器上的传统 WAR 文件运行 SpringApplication。这个类将 Servlet、Filter 和 ServletContextInitializer bean从应用程序上下文绑定到服务器。

扩展 SpringBootServletInitializer 类还允许通过覆盖 configure()方法来配置 Servlet 容器运行时的应用程序。

### 2. SpringBootServletInitializer

为了更实用，展示一个扩展 Initializer 类的主类的示例。

名为 WarInitializerApplication 的 @SpringBootApplication 类扩展了 SpringBootServletInitializer 并覆盖了 configure() 方法。 该方法使用 SpringApplicationBuilder 简单地将自定义的类注册为应用程序的配置类：

```java
@SpringBootApplication
public class WarInitializerApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(WarInitializerApplication.class);
    }
    
    public static void main(String[] args) {
        SpringApplication sa = new SpringApplication(
          WarInitializerApplication.class);
        sa.run(args);
    }
    
    @RestController
    public static class WarInitializerController {
        @GetMapping("/")
        public String handler() {
           // ...
        }
    }
} 
```

现在，如果将应用程序打包为WAR，将能够以传统方式将其部署在任何Web容器上，并且将执行在configure()方法中添加的逻辑。

如果想将它打包为JAR文件，那么需要向main()方法添加相同的逻辑，以便嵌入式容器也可以获取。