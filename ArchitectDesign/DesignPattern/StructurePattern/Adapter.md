# 适配器模式

原文：http://c.biancheng.net/view/1361.html



## 1. 定义和特点

​        适配器模式的定义：将一个类的接口转换成客户希望的另一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。

​        适配器模式分为**类结构型模式**和**对象结构型模式**两种。前者类之间的耦合度比后者高，且要求程序员了解现有组件库中的相关组件的内部结构，所以应用比较少。

​        该模式的主要优点有：

* 客户端通过适配器可以透明地调用目标接口
* 复用了现存的类，程序员不需要修改原有代码，而重用现有的适配者类
* 将目标类和适配者类解耦，解决了目标类和适配者类接口不一致的问题
* 在很多业务场景中符合开闭原则



​        缺点也有如下几点：

* 适配器编写过程需要结合业务场景全面考虑，可能会增加系统的复杂度
* 增加代码阅读难度，降低代码可读性，过多使用适配器会使系统代码变得凌乱

## 2. 结构与实现

​        **类适配器模式**可以采用多重继承方式实现，如C++可以定义一个适配器类来同时继承当前系统的业务接口和现有组件库中已经存在的组件接口；Java不支持多重继承，但可以定一个适配器类来实现当前系统的业务接口，同时又继承现有组件库中已经存在的组件。

​        **对象适配器模式**可采用将现有组件库中已经实现的组件引入适配器类中，该类同时实现当前系统的业务接口。

### 2.1 模式的结构

​        适配器模式（Adapter）包含以下主要角色：

1. 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口
2. 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口
3. 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者

​        **类适配器模式**的结构图如下：

![1](../../images/StructurePattern/Adapter/1.gif)

​        **对象适配器模式**的结构图：

![2](../../images/StructurePattern/Adapter/2.gif)

### 2.2 模式的实现

#### 2.2.1 类适配器模式的代码

```java
package adapter;
//目标接口
interface Target
{
    public void request();
}
//适配者接口
class Adaptee
{
    public void specificRequest()
    {       
        System.out.println("适配者中的业务代码被调用！");
    }
}
//类适配器类
class ClassAdapter extends Adaptee implements Target
{
    public void request()
    {
        specificRequest();
    }
}
//客户端代码
public class ClassAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("类适配器模式测试：");
        Target target = new ClassAdapter();
        target.request();
    }
}
```

程序的运行结果如下：

```
类适配器模式测试：
适配者中的业务代码被调用！
```



#### 2.2.2 对象适配器模式的代码

```java
package adapter;
//对象适配器类
class ObjectAdapter implements Target
{
    private Adaptee adaptee;
    public ObjectAdapter(Adaptee adaptee)
    {
        this.adaptee=adaptee;
    }
    public void request()
    {
        adaptee.specificRequest();
    }
}
//客户端代码
public class ObjectAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("对象适配器模式测试：");
        Adaptee adaptee = new Adaptee();
        Target target = new ObjectAdapter(adaptee);
        target.request();
    }
}
```

说明：对象适配器模式中的“目标接口”和“适配者类”的代码同类适配器模式一样，只要修改适配器类和客户端的代码即可。

运行结果：

```
对象适配器模式测试：
适配者中的业务代码被调用！
```

## 3. 应用实例

​        【例1】用适配器模式（Adapter）模拟新能源汽车的发动机。

​        分析：新能源汽车的发动机有电能发动机（Electric Motor）和光能发动机（Optical Motor）等，各种发动机的驱动方法不同，例如，电能发动机的驱动方法 electricDrive() 是用电能驱动，而光能发动机的驱动方法 opticalDrive() 是用光能驱动，它们是适配器模式中被访问的适配者。

​        客户端希望用统一的发动机驱动方法 drive() 访问这两种发动机，所以必须定义一个统一的目标接口 Motor，然后再定义电能适配器（Electric Adapter）和光能适配器（Optical Adapter）去适配这两种发动机。

​        客户端想访问的新能源发动机的适配器的名称放在 XML 配置文件中，客户端可以通过对象生成器类 ReadXML 去读取。这样，客户端就可以通过 Motor 接口随便使用任意一种新能源发动机去驱动汽车，下图 所示是其结构图。

![3](../../images/StructurePattern/Adapter/3.gif)

​        代码如下：

```java
package adapter;
//目标：发动机
interface Motor
{
    public void drive();
}
//适配者1：电能发动机
class ElectricMotor
{
    public void electricDrive()
    {
        System.out.println("电能发动机驱动汽车！");
    }
}
//适配者2：光能发动机
class OpticalMotor
{
    public void opticalDrive()
    {
        System.out.println("光能发动机驱动汽车！");
    }
}
//电能适配器
class ElectricAdapter implements Motor
{
    private ElectricMotor emotor;
    public ElectricAdapter()
    {
        emotor=new ElectricMotor();
    }
    public void drive()
    {
        emotor.electricDrive();
    }
}
//光能适配器
class OpticalAdapter implements Motor
{
    private OpticalMotor omotor;
    public OpticalAdapter()
    {
        omotor=new OpticalMotor();
    }
    public void drive()
    {
        omotor.opticalDrive();
    }
}
//客户端代码
public class MotorAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("适配器模式测试：");
        Motor motor=(Motor)ReadXML.getObject();
        motor.drive();
    }
}
```

```java
package adapter;
import javax.xml.parsers.*;
import org.w3c.dom.*;
import java.io.*;
class ReadXML
{
    public static Object getObject()
    {
        try
        {
            DocumentBuilderFactory dFactory=DocumentBuilderFactory.newInstance();
            DocumentBuilder builder=dFactory.newDocumentBuilder();
            Document doc;                           
            doc=builder.parse(new File("src/adapter/config.xml"));
            NodeList nl=doc.getElementsByTagName("className");
            Node classNode=nl.item(0).getFirstChild();
            String cName="adapter."+classNode.getNodeValue();
            Class<?> c=Class.forName(cName);
              Object obj=c.newInstance();
            return obj;
         }  
         catch(Exception e)
         {
                   e.printStackTrace();
                   return null;
         }
    }
}
```

```XML
?xml version="1.0" encoding="UTF-8"?>
<config>
	<className>ElectricAdapter</className>
</config>
```

运行结果：

```
适配器模式测试：
电能发动机驱动汽车！
```

注意：如果将配置文件中的 ElectricAdapter 改为 OpticalAdapter，则运行结果如下：

```
适配器模式测试：
光能发动机驱动汽车！
```

## 4. 应用场景

​         适配器模式（Adapter）通常用于以下场景：

* 以前开发的系统存在满足新系统功能需求的类，但其接口同新系统的接口不一致
* 使用第三方提供的组件，但组件的接口定义和自己的接口定义不同

## 5. 模式扩展

​        适配器模式（Adapter）可扩展为双向适配器模式，双向适配器类既可以把适配者接口转换成目标接口，也可以把目标接口转换成适配者接口，其结构如下图：

![4](../../images/StructurePattern/Adapter/4.gif)

代码如下：

```

```



```java
package adapter;
//目标接口
interface TwoWayTarget
{
    public void request();
}
//适配者接口
interface TwoWayAdaptee
{
    public void specificRequest();
}
//目标实现
class TargetRealize implements TwoWayTarget
{
    public void request()
    {       
        System.out.println("目标代码被调用！");
    }
}
//适配者实现
class AdapteeRealize implements TwoWayAdaptee
{
    public void specificRequest()
    {       
        System.out.println("适配者代码被调用！");
    }
}
//双向适配器
class TwoWayAdapter  implements TwoWayTarget,TwoWayAdaptee
{
    private TwoWayTarget target;
    private TwoWayAdaptee adaptee;
    public TwoWayAdapter(TwoWayTarget target)
    {
        this.target=target;
    }
    public TwoWayAdapter(TwoWayAdaptee adaptee)
    {
        this.adaptee=adaptee;
    }
    public void request()
    {
        adaptee.specificRequest();
    }
    public void specificRequest()
    {       
        target.request();
    }
}
//客户端代码
public class TwoWayAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("目标通过双向适配器访问适配者：");
        TwoWayAdaptee adaptee=new AdapteeRealize();
        TwoWayTarget target=new TwoWayAdapter(adaptee);
        target.request();
        System.out.println("-------------------");
        System.out.println("适配者通过双向适配器访问目标：");
        target=new TargetRealize();
        adaptee=new TwoWayAdapter(target);
        adaptee.specificRequest();
    }
}
```

程序的运行结果如下：

```
目标通过双向适配器访问适配者：
适配者代码被调用！
-------------------
适配者通过双向适配器访问目标：
目标代码被调用！
```

