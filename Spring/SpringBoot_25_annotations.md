# SpringBoot的25个注解

原文：https://www.cnblogs.com/xiaobug/p/11438904.html



### 1. @SpringBootApplication

​        这是 Spring Boot 最最核心的注解了。用在 SpringBoot的主类上，标识这是一个 Spring Boot 应用，用来开启Spring Boot 的各项能力。

​        其实，这个注解就是 ***@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan*** 这三个注解的组合，也可以用这三个注解来替代 @SpringBootApplication。

### 2. @EnableAutoConfiguration

​        允许 SpringBoot 自动配置注解，开启这个注解之后，SpringBoot就能根据当前类路径下的包或者类来配置Spring Bean。

​        如：当前类路径下有 MyBatis 这个 jar 包，MybatisAutoConfiguration 注解就能根据相关参数来配置 MyBatis 的各个 SpringBean。

### 3. @Configuration

​        这个注解是 Spring3.0 添加的，用来替代 applicationContext.xml 配置文件，所有这个配置文件能做到的事情，都可以通过这个注解所在类来进行注册。

​        下面几个相关注解也是非常重要的。

#### 3.1 @Bean

​        用来代替 XML 配置文件里的 *<bean ...>* 配置。

#### 3.2 @Import

​        这是 Spring 3.0 添加的新注解，**用来导入一个或者多个 @Configuration 注解修饰的类**，这在 SpringBoot 里应用很多。【 *注意，与 @ImportResource 的区别* 】

#### 3.3 @ImportResource

​        这是 Spring 3.0 添加的新注解，**用来导入一个或者多个 Spring 配置文件**，这对 Spring Boot 兼容老项目非常有用，因为有些配置无法通过 Java Config 的形式来配置，就只能用这个注解来导入（导入传统的xml配置文件）。

### 4. @SpringBootConfiguration

​        这个注解是 @Configuration 注解的变体，只是用来修饰 SpringBoot 配置而已，或者有利于 SpringBoot的后续扩展。引用了 @Configuration 注解。源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {    
}
```

### 5. @ComponentScan

​        这是 Spring 3.1 添加的一个注解，用来替代配置文件中的 *component-scan* 配置，开启组件扫描，即自动扫描包路径下的 @Component 注解进行注册 bean 实例到 context 中。

### 6. @Conditional

​     是 Spring 4.0 添加的新注解，用来标识一个 SpringBean 或者 Configuration 配置文件，当满足指定条件才开启配置。

### 7. @ConditionalOnBean

​        组合 @Conditional 注解，当容器中有指定的 Bean 才开启配置

### 8. @ConditionalOnMissingBean

​        组合 @Conditional 注解，和 @ConditionalOnBean 注解相反，当容器中没有指定的 Bean 时才开启配置。

### 9. @ConditionalOnClass

​        组合 @Conditional 注解，当容器中有指定的 Class 才开启配置

### 10. @ConditionalOnMissingClass

​        组合 @Conditional 注解，和 @ConditionalOnClass 相反，当容器中没有指定的 Class 才开启配置。

### 11. @ConditionalOnWebApplication

​        组合 @Conditional 注解，当前项目类型是 Web 项目才开启配置

​       当前项目有以下 3 种类型：

```java
enum Type {
    // Any web application will match
    ANY,
    // Only servlet-based web application will match
    SERVLET,
    // Only reactive-based web application will match
    REACTIVE
}
```

### 12. @ConditionalOnNotWebApplication

​        组合 @Conditional 注解，和 @ConditionalOnWebApplication 注解相反，当前项目类型不是 WEB 项目才开启配置。

### 13. @ConditionalOnProperty

​        组合 @Conditional 注解，当指定的属性有指定的值时，才开启配置。

#### 14. @ConditionalOnExpression

​        组合 @Conditional 注解，当 SpEL 表达式为 true 时才开启配置。

### 15. @ConditionalOnJava

​        组合 @Conditional 注解，当运行的 JVM 在指定的版本范围时才开启配置。

### 16. @ConditionalOnResource

​        组合 @Conditional 注解，当类路径下有指定的资源才开启配置。

### 17. @ConditionalOnJndi

​        组合 @Conditional 注解，当指定的 JNDI 存在时才开启配置。

### 18. @ConditionalOnCloudPlatform

​        组合 @Conditional 注解，当指定的云平台激活时才开启配置。

### 19. @ConditionalOnSingleCandidate

​        组合 @Conditional 注解，当指定的 class 在容器中只有一个 Bean，或者同时有多个但为首选时才开启配置。

### 20. @ConfigurationProperties

​        用来加载额外的配置（如 .properties 文件），可用在 @Configuration 注解类，或者 @Bean 注解方法上面。

### 21. @EnableConfigurationProperties

​        一般要配合 @ConfigurationProperties 注解使用，用来开启 @ConfigurationProperties 注解配置 Bean 的支持。

### 22. @AutoConfigureAfter

​        用在自动配置类上面，表示该自动配置类**需要在另外指定的自动配置类配置完之后**。

​        如： MyBaties 的自动配置类，需要在数据源自动配置类之后：

```java
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration{
    // ....
}
```

### 23. @AutoConfigureBefore

​        这个和 @AutoConfigureAfter 注解使用相反，表示该自动配置类需要**在另外指定的自动配置类配置完成之前**。



