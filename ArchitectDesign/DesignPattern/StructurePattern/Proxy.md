# 代理模式

原文：http://c.biancheng.net/view/1359.html



## 1. 定义与特点

代理模式的定义：由于某些原因，需要给某个对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。

代理模式主要有以下几个优点：

* 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用
* 代理对象可以扩展目标对象的功能
* 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了系统的可扩展性

​        

也有如下一些缺点：

* 代理模式会造成系统设计中类的数量增加
* 在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢
* 增加了系统的复杂度

注：通过***动态代理***可以解决上面提到的“缺点”。

## 2. 结构与实现

代理模式结构比较简单，主要是通过定义一个继承抽象主题的代理来包含真实主题，从而实现对真实主题的访问。下面来分析其基本结构和实现方法。

### 2.1 模式的结构

代理模式的主要角色如下：

1. 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法
2. 真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象
3. 代理（Proxy）类：提**供了与真实主题相同的接口**，其**内部含有对真实主题的引用**，它可以访问、控制或者扩展真实主题的功能

其结构图如下所示：

![1](../../images/DesignPattern/Structure/1.gif)

在代码中，一般代理被理解为代码增强，实际上就是在原代码逻辑前后增加一些代码逻辑，而使调用者无感知。

根据代理的创建时期，代理模式分为**静态代理**和**动态代理**：

* 静态：由程序员创建代理类，或者特定工具自动生成源代码在对其编译，在运行前，代理类的.class文件已经存在
* 动态：在程序运行时，运用反射机制动态创建而成

### 2.2 实现方法

代理模式的实现代码如下：

```java
package proxy;

public class ProxyTest {
    public static void main(String[] args) {
        Proxy proxy = new Proxy();
        proxy.Request();
    }
}

//抽象主题
interface Subject {
    void Request();
}

//真实主题
class RealSubject implements Subject {
    public void Request() {
        System.out.println("访问真实主题方法...");
    }
}

//代理
class Proxy implements Subject {
    private RealSubject realSubject;

    public void Request() {
        if (realSubject == null) {
            realSubject = new RealSubject();
        }
        preRequest();
        realSubject.Request();
        postRequest();
    }

    public void preRequest() {
        System.out.println("访问真实主题之前的预处理。");
    }

    public void postRequest() {
        System.out.println("访问真实主题之后的后续处理。");
    }
}
```

运行结果：

```
访问真实主题之前的预处理。
访问真实主题方法...
访问真实主题之后的后续处理。
```

## 3. 应用实例

【例】： 韶关“天街e角”公司是一家婺源特产公司的代理公司，用代理模式实现

分析：本实例中的“婺源特产公司”经营许多婺源特产，它是真实主题，提供了显示特产的 display() 方法，可以用窗体程序实现。而韶关“天街e角”公司是婺源特产公司特产的代理，通过调用婺源特产公司的 display() 方法显示代理产品，当然它可以增加一些额外的处理，如包裝或加价等。客户可通过“天街e角”代理公司间接访问“婺源特产公司”的产品，下图所示是公司的结构图：

![2](../../images/DesignPattern/Structure/2.gif)

代码如下：

```java
package proxy;

import java.awt.*;
import javax.swing.*;

public class WySpecialtyProxy {
    public static void main(String[] args) {
        SgProxy proxy = new SgProxy();
        proxy.display();
    }
}

//抽象主题：特产
interface Specialty {
    void display();
}

//真实主题：婺源特产
class WySpecialty extends JFrame implements Specialty {
    private static final long serialVersionUID = 1L;

    public WySpecialty() {
        super("韶关代理婺源特产测试");
        this.setLayout(new GridLayout(1, 1));
        JLabel l1 = new JLabel(new ImageIcon("src/proxy/WuyuanSpecialty.jpg"));
        this.add(l1);
        this.pack();
        this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }

    public void display() {
        this.setVisible(true);
    }
}

//代理：韶关代理
class SgProxy implements Specialty {
    private WySpecialty realSubject = new WySpecialty();

    public void display() {
        preRequest();
        realSubject.display();
        postRequest();
    }

    public void preRequest() {
        System.out.println("韶关代理婺源特产开始。");
    }

    public void postRequest() {
        System.out.println("韶关代理婺源特产结束。");
    }
}
```

程序运行结果如图所示：

![3](../../images/DesignPattern/Structure/3.jpg)



## 4. 用场景

当无法或不想直接引用某个对象或访问某个对象存在困难时，可以通过代理对象来间接访问。使用代理模式主要有两个目的：一是保护目标对象，二是增强目标对象。

前面分析了代理模式的结构与特点，现在来分析以下的应用场景。

* 远程代理：这种方式通常是为了隐藏目标对象存在于不同地址空间的事实，方便客户访问。例如，用户申请某些网盘空间时，会在用户的文件系统中建立一个虚拟的硬盘，用户访问虚拟硬盘时实际上访问的是网络空间
* 虚拟代理：这种方式通常用于要创建的目标对象开销很大时。例如：下载一副很大的图像需要很长时间，因某种计算比较复杂而短时间内无法完成，这时就可以先用小比例的虚拟代理替代真实的对象，消除用户对服务器慢的感觉
* 安全代理：这种方式通常用于控制不同种类客户对真实对象的访问权限
* 智能指引：主要用于调用目标对象时，代理附加一些额外的处理功能。例如，增加计算真实对象的引用次数的功能，这样当该对象没有被引用时，就可以自动释放它
* 延迟加载：为了提高系统的性能，延迟对目标的加载。例如，Hibernate中就存在属性的延迟加载和关联表的延迟加载

## 5. 扩展

在前面介绍的代理模式中，代理类中包含了对真实主题的引用，这种方式存在两个缺点。

1. 真实主题与代理主题一一对应，增加真实主题也要增加代理
2. 设计代理以前，真实主题必须事先存在，不太灵活。采用动态代理模式可以解决上述问题。如：Spring AOP，其结构图如下：

![4](../../images/DesignPattern/Structure/4.gif)