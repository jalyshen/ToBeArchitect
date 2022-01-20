# Java注解的基本原理

原文：https://blog.csdn.net/weixin_43994338/article/details/106014180?share_token=95198D94-6A02-4A5F-8795-A6FAD7AB63A7&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1



## 一. 什么是 Java 注解

在注解（Annotation）出现之前，XML 配置以其松耦合的优势一度称为各大框架的主流配置方式，但是随着项目规模越来越大，XML 配置也越来越复杂，维护成本原来越高。于是有人提出了一种标记式高耦合的配置方式--注解（Annotation）。注解说到底，不过是一种特殊的注释而已。如果没有解析它的代码，可能它连注释都不如。

*问题：解释这些注解的代码在哪儿呢？*

## 二. 元注解

元注解，就是用于修饰注解的注解，用在注解定义上。如 @Target， @Retention，一般用于指定标记的注解的生命周期等一些信息。例如：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

Java 中有如下几个元注解：

* @Target：注解的作用目标
* @Retention：注解的生命周期
* @Documented：注解是否应当被包含在 JavaDoc 文档中
* @Inherited：是否允许子类继承该注解

@Target注解属性值为 ElementType ，是一个枚举类型，有如下的取值：

* ElementType.TYPE：允许被修饰的注解作用在类、接口和枚举上
* ElementType.FIELD：允许作用在属性字段上
* ElementType.METHOD：允许作用在方法上
* ElementType.PARAMETER：允许作用在方法参数上
* ElementType.CONSTRUCTOR：允许作用在构造器上
* ElementType.LOCAL_VARIABLE：允许作用在本地局部变量上
* ElementType.ANNOTATION_TYPE：允许作用在注解上
* ElementType.PACKAGE：允许作用在包上

@Retention 注解属性可选的值如下：

* RetentitionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
* RetentitionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
* RetentitionPolicy.RUNTIME：永久保存，可以反射获取

## 三. 注解原理

注解，本质上是一个继承了 Annotation 的特殊接口，用于解析注解的类是 Java 运行时生成的动态代理类。而通过反射获取注解时，返回的是 Java 运行时生成的动态代理对象 $Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用 ***AnnotatonInvocationHandler*** 的 invoke 方法。该方法会从 memberValues 这个 Map 中索引处对应的值。而 memberValues 的来源是 Java 常量池。

## 四. 自定义注解

自定义注解的一些要求：

* 形式为 @interface，自定义的注解都会自动继承 java.lang.Annotation 这个接口
* 访问修饰符只能是 public 或者 default

### 4.1 代码实现

* PeopleName.java

  ```java
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface PeopleName {
      String value() default "";
  }
  ```

  

* PeopleColor.java

  ```java
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface PeopleColor {
      
      public enum Color {
          YELLOW, WHITE, BLACK    
      }
      
      Color peopleColor() default Color.YELLOW;
  }
  ```

  

* ResolveMyAnnotation.java

  ```java
  public class ResolveMyAnnotation {
      public static void getPeopleInfo(Class<?> peopleClass) {
          String strPeopleName = "人的姓名：";
          String strPeopleColor = "人的肤色：";
          Field[] fields = peopleClass.getDeclaredField();
          for (Feild field : fields) {
              if (field.isAnnotationPresent(PeopleName.class)) {
                  PeopleName peopleName = field.getAnnotation(PeopleName.class);
                  strPeopleName=strPeopleName + peopleName.value();
                  System.out.println(strPeopleName);
              }
              if (field.isAnnotationPresent(PeopleColor.class)) {
                  PeopleColor peopleColor = field.getAnnotation(PeopleColor.class);
                 strPeopleColor=strPeopleColor + peopleColor.peopleColor();
                  System.out.println(strPeopleColor);
              }
          }
      }
  }
  ```

  

* People.java

  ```java
  /**
   * 注解使用
   */
  public class People {
      @PeopleName("张三")
      private String peopleName;
      @PeopleColor(peopleColor = PeopleColor.Color.BLACK)
      private String peopleColor;
  
      public String getPeopleName() {
          return peopleName;
      }
  
      public void setPeopleName(String peopleName) {
          this.peopleName = peopleName;
      }
  
      public String getPeopleColor() {
          return peopleColor;
      }
  
      public void setPeopleColor(String peopleColor) {
          this.peopleColor = peopleColor;
      }
  }
  ```

  

* Test.java

  ```java
  /**
   * 测试
   */
  public class Test {
      public static void main(String[] args){
          ResolveMyAnnotation.getPeopleInfo(People.class);
      }
  }
  ```

  

* 测试结果

  ```shell
  人的姓名：张三
  人的肤色：BLACK
  ```

  