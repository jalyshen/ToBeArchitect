# Java Transient 关键字使用

原文：https://www.cnblogs.com/lanxuezaipiao/p/3369962.html



## 一. transient 的作用与用法

一个对象只要实现了 ***java.io.Serializable*** 接口，这个对象就可以被序列化。Java 的这种序列化模式，使得开发人员不必关心对象是如何被序列化的，就会把类的**所有**<font color='red'>属性和方法</font>都自动序列化了。

但是在实际开发过程中，常常遇到这样的情况：有些属性需要序列化，而有些则不需要。例如：某个用户的一些敏感信息（密码、银行卡号等），为了安全，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上 ***transient***  关键字。加上之后，这些字段的生命周期仅存于调用者的内存中，而不会被写到磁盘或者网络里。

## 二. transient 使用小结

* 一旦变量被 **transient** 修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问
* **transient** 关键字**只能**修饰变量，不能修饰方法和类
* **transient** **不能**修饰**本地变量**（就是局部变量）
* 被 transient 修饰的变量不再能被序列化
* 静态变量（类变量）不管是否被 transient 修饰，都**不会**被序列化

## 三. 实现序列化的方式

1. 实现接口  ***java.io.Serializable***，该类会自动进行序列化

2. 实现接口 ***java.io.Externalizable***，需要开发人员自己实现两个接口

   1. ```java
      void writeExternal(ObjectOutput out) throws IOException;
      ```

   2. ```java
      void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
      ```

   如果使用这个方式，则被 transient 关键字的字段如果在这两个方法内，也会被序列化。