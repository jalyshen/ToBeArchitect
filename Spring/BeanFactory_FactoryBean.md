# BeanFactory 和 FactoryBean 的区别 

原文：https://www.toutiao.com/a6796880085057012228/



**BeanFactory** 是 Spring 中**比较原始的Factory**。如 XMLBeanFactory 就是一种典型的BeanFactory。原始的BeanFactory无法支持Spring的许多插件，如AOP功能、Web应用等。

**ApplicationContext** 接口，它由 **BeanFactory** 接口派生而来。ApplicationContext 包含 BeanFactory 的所有功能，通常建议比 BeanFactory 优先使用。

## 一. BeanFactory和 FactoryBean的区别

**BeanFactory 是接口，提供了 IoC 容器的最基本形式**，给具体的 IoC 容器的实现提供了规范。

**FactoryBean 也是接口，为 IoC 容器中的Bean的实现（实例化）提供了更加灵活的方式**。FactoryBean 在 IoC 容器的基础上给Bean 的实现（实例化）加上了一个简单工厂模式和装饰模式，可以在 getObject() 方法中灵活配置。其实，在Spring源码中有很多FactoryBean的实现类。

* **区别**：BeanFactory 是一个 Factory，也就是 IoC 容器或者对象工厂；FactoryBean 是一个 Bean。在 Spring 中，所有的 Bean 都是由 BeanFactory（也就是 IoC 容器）来进行管理的。
* 但是对于 FactoryBean 而言，这个 Bean 不是简单的Bean，而是一个能生产或者修改对象生成的工厂 Bean，它的实现和设计模式中的工厂模式和装饰器模式类似。

### 1.1 BeanFactory

BeanFactory，以 **Factory** 结尾，表示它是一个工厂类（接口），它是负责生产和管理Bean的一个工厂。在 Spring 中，**BeanFactory 是 IoC 容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象以及建立这些对象间的依赖**。

BeanFactory 只是一个接口，并不是 IoC 容器的具体实现，但是 Spring 容器给出了很多种实现，如：DefaultListableBeanFactory、XmlBeanFacotory、ApplicationContext等，其中 XmlBeanFactory 就是常用的一个，该实现将以 XML 方式描述组成应用的对象以及对象间的依赖关系。XmlBeanFactory 类将持有此 XML 配置元数据，并用它来构建一个完全可以配置的系统或者应用。

都是附加了某种功能的实现。它为其他具体的 IoC 容器提供了最基本的规范，例如：DefaultListableBeanFactory，XmlBeanFactory，ApplicationContext 等具体的容器都是实现了 BeanFactory，再在其基础上附加了其他的功能。

BeanFactory 和 ApplicationContext 就是 Spring 框架的两个 IoC 容器，现在一般使用 ApplicationContext，其不但包含了 BeanFactory 的所有功能（作用），同时还进行了更多的扩展。

原始的 BeanFactory 无法支持 Spring 的许多插件，如 AOP 功能、Web引用等。而ApplicationContext 继承 BeanFactory，并且以一种更面向框架的方式工作，以及对上下文进行分层和实现继承，它提供了以下的功能：

* MessageSource，提供国际化的消息访问
* 资源访问，如 URI 和文件
* 事件传播
* 载入多个（有继承关系）上下文，使得每一个上下文都专注于一个特定的层次，比如应用的 Web 层

在不使用 Spring 框架前，通常在 servcie 层中要使用 DAO 层的对象，不得不在 Service 层中 new 一个对象，这样就存在一个问题：层与层之间的依赖。Service 层要用 DAO 层对象，需要配置到 XML 文件中去，至于对象是如何创建的，关系是怎么组合的，都交给了 Spring 框架去实现。一般有以下三种实现方法：

* 方法一

  ```java
  Resource resource = new FileSystemResource("beans.xml");
  BeanFactory factory = new XmlBeanFactory(resource);
  ```

  

* 方法二

  ```java
  ClassPathResource resource = new ClassPathResource("beans.xml");
  BeanFactory factory = new XmlBeanFactory(resource);
  ```

  

* 方法三

  ```java
  ApplicationContext context = new ClassPathXmlApplicationContext(new  String[]{"applicatonContext.xml","applicationContext-part2.xml"});
  BeanFactory factory = (BeanFactory) context;
  ```

基本就是这些了，接着使用 ***getBean(String beanName)*** 方法就可以取得 bean 的实例；BeanFactory 提供的方法及极其简单，仅提供了六种方法供客户调用：

* **boolean containsBean(String name)**: 判断工厂中是否包含给定名称的 bean 的定义，若有，则返回 true；否则返回 false
* **Object getBean(String name)**: 返回给定名称注册的 bean 实例。根据 bena 的配置情况，如果是 singleton 模式将返回一个共享实例，否则返回一个新建的实例。如果没有找到指定的 bean，该方法可能会抛出一个异常
* **Object getBean(String name, Class clazz)**: 返回以给定名称注册的 bean 实例，并转换为给定的 class 类型
* **Class getType(String name)**: 返回给定名称的 bean 的 Class，如果没有找到指定的 bean 实例，则抛出 NoSuchBeanDefinitionException 异常
* **boolean isSingleton(String name)**: 判断给定名称的 bean 定义是否为单例模式
* **String[] getAilases(String name)**: 返回给定 bean 名称的所有别名

### 1.2 FactoryBean

一般情况下，Spring 通过**反射机制**利用 <bean> 的 class 属性指定实现类实例化 Bean，在某些情况下，实例化 Bean 过程比较复杂，如果按照传统的方式，则需要在 <bean> 中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。

Spring 为此提供了一个 **FactoryBean 的工厂类接口**，用户可以通过**实现**该接口**定制实例化 Bean 的逻辑**。FactoryBean 接口对于 Spring 框架来说，占据重要的地位，Spring 本身就提供了 70 多个 FactoryBean 的实现。它们隐藏了实例化一些复杂 Bean 的细节，给上层应用带来了便利。从 Spring 3.0 开始，FactoryBean 开始支持泛型，即接口声明改成了 ***FactoryBean<T>*** 的形式。

FactoryBean 以 Bean 结尾，它表示一个 Bean，但又不同于普通的Bean：它是实现了FactoryBean<T> 接口的 Bean，根据该 Bean 的 ID 从 BeanFactory 中获取的实际上是 FactoryBean 的 ***getObject()*** 返回的对象，而不是 FactoryBean 本身。如果要获取 FactoryBean 对象，请在 ID 前面加一个 **&** 符号来获取。

例如，自己实现了一个 FactoryBean，功能如下：用来代理一个对象，对该对象的所有方法做一个拦截，在调用前后都输出一行log，模仿 ProxyFactoyBean 的功能：

```java
/**
 * my factory bean<p>
 * 代理一个类，拦截该类的所有方法，在方法的调用前后进行日志的输出
 * @author daniel.zhao
 *
 */
public class MyFactoryBean implements BeanFactory<Object>, 
                                      InitializingBean, DisposableBean 
{
  private static final Logger logger = 
          LoggerFactory.getLogger(MyFactoryBean.class);

  private String interfaceName;
  private Object target;
  private Object proxyObj;

  @Override
  public void destory() throws Exception {
      logger.debug("destroy....");
  }

  @Override
  public void aftetPropertiesSet() throws Exception {
      proxyObj = Proxy.newProxyInstance(
        this.getClass().getClassLoader(),
        new Class[] { class.forName(interfaceName)},
        new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, 
                                 Method methohd, 
                                 Object[] args) throws Throwable {
                logger.debug("invoke method..." + method.getName());
                logger.debug("invoke method before..." + 
                             System.currentTimeMillis());
                
                Obejct result = method.invoke(target, args);
                logger.debug("invoke method after..." + 
                             SystemcurrentTimeMillis());
                
                return result;
            }
        }
      );
      logger.debug("after PropertiesSet...");
  }

  @Override
  public Object getObject() throws Exception {
      logger.debug("getObject...");
      return proxyObj;
  }

  @Override
  public Class<?> getObjectType() {
      return proxyObj == null ? object.class : proxyObj.getClass();
  }
                                          
  @Override
  public boolean isSingleton() {
    return true;      
  }

  public String getInterfaceName() {
      return interfaceName;
  }

  public void setInterfaceName(String interfaceName) {
      return this.interfaceName = interfaceName;
  }

  public Object getTarget() {
      return target;
  }

  public void setTarget(Object target) {
      return this.target = target;
  }

  public Object getProxyObj() {
      return proxyObj;
  }

  public void setProxyObj(Object proxyObj) {
      return this.proxyObj = proxyObj;
  }
}
```

XML-Bean 的配置如下：

```xml
<bean id="fbHelloWorldService" class="com.ebao.xxx.MyFactoryBean">
   <property name="interfaceName" 
             value="com.ebao.xxx.HelloWorldService" />
   <property name="target" ref="helloWorldService" />
</bean>
```

下面做一下测试：

```java
@RunWith(JUnit4ClassRunner.class)
@ContextConfiguration(classes = { MyFactoryBeanConfig.class })
public class MyFactoryBeanTest {
    @Autowired
    private ApplicationContext context;    
    /**
     * 测试验证FactoryBean原理，代理一个servcie在调用其方法的前后，
     * 打印日志亦可作其他处理
     * 从ApplicationContext中获取自定义的FactoryBean
     * context.getBean(String beanName) ---> 最终获取到
     *       的Object是FactoryBean.getObejct(), 
     * 使用Proxy.newInstance生成service的代理类
     */
    @Test
    public void testFactoryBean() {
      HelloWorldService helloWorldService = 
         (HelloWorldService) context.getBean("fbHelloWorldService");
      helloWorldService.getBeanName();
      helloWorldService.sayHello();
    }
}
```

FactoryBea 是一个接口，当在 IoC 容器中的 Bean 实现了 FactoryBean 后，通过 getBean(String beanName) 获取到的 Bean 对象并不是 FactoryBean 的实现类对象，而是这个实现类中的 ***getObject()*** 方法返回的对象。要想获得 FactoryBean 的实现类，就要 ***getBean(&BeanName)***，在 beanName之前加上 <font color='red'>***&***</font>。

FactoryBean 接口如下：

```java
package org.springframework.beans.factory;  
public interface FactoryBean<T> {  
    T getObject() throws Exception;  
    Class<?> getObjectType();  
    boolean isSingleton();  
}
```

这三个接口分表表示：

* **T getObject()** :返回由 FactoryBean 创建的 Bean 实例，如果 isSingleton() 返回 true，则该实例会放到 Spring 容器中单实例的缓存池中
* **boolean isSingleton()**  : 返回由 FactoryBean 创建的 Bean 实例的作用域是 singletone 还是 prototype
* **class<T> getObjectType()** :返回 FactoryBean 创建的 Bean 类型



当配置文件中 <bean> 的 class 属性配置的实现类是 FactoryBean 时，通过 ***getBean()*** 方法返回的不是 FactoryBean 本身，而是 *FactoryBean#getObject()* 方法所返回的对象，相当于 *FactoryBean#getObject()* 代理了 getBean() 方法。

例如：如果使用传统方法配置下面的 Car 对象的 <bean> 时， Car 的每个属性分别对应一个 <property> 元素标签：

```java
public   class  Car  {  
  private  int    maxSpeed ;  
  private  String brand ;  
  private  double price ;  
        
  public int getMaxSpeed(){  
    return this.maxSpeed;  
  }  

  public void  setMaxSpeed (int  maxSpeed )   {  
    this.maxSpeed  = maxSpeed;  
  }  
    
  public String getBrand() {  
    return   this.brand ;  
  }  
        
  public void  setBrand ( String brand ) {  
    this.brand  = brand;  
  }  
        
  public double  getPrice () {  
    return   this.price ;  
  }  
    
  public void  setPrice (double  price) {  
    this.price  = price;  
  }  
}
```

如果用 FactoryBean 的方式实现就灵活点儿，下面通过逗号分隔符的方式一次性的为 Car 的所有属性指定配置值：

```java
import  org.springframework.beans.factory.FactoryBean;  
public class  CarFactoryBean  implements  FactoryBean<Car>  {  
    private  String carInfo;  
    public  Car getObject()   throws  Exception{  
        Car car =  new Car();  
        String []  infos = carInfo.split ( "," ) ;  
        car.setBrand(infos [0]);  
        car.setMaxSpeed(Integer.valueOf(infos[1]));
        car.setPrice(Double.valueOf(infos[2])) ;  
        return car;  
    }  
    public Class<Car> getObjectType(){  
        return Car.class ;  
    }  
    public boolean  isSingleton(){  
        return false ;  
    }  
    public String getCarInfo(){  
        return this.carInfo ;  
    }  

    // 接受逗号分割符设置属性信息  
    public void setCarInfo(String carInfo){
        this.carInfo = carInfo;  
    }  
}
```

有了这个 CarFactoryBean 之后，就可以在配置文件中使用下main这种自定义的配置方式配置CarBean了：

```xml
<bean d="car"class="com.baobaotao.factorybean.CarFactoryBean"
        P:carInfo="法拉利,400,2000000"/>
```

当调用 getBean("car") 时，Spring 通过反射机制发现 CarFactoryBean 实现了 FactoryBean 的接口，这时 Spring 容器就调用接口方法 CarFactoryBean#getObject() 方法返回。如果希望获取 CarFactoryBean 实例，则需要在使用 getBean(beanName) 方法时在 beanName 前**显式**的加上"**&**"前缀，如： getBean("&car");

下面是一个应用 FactoryBean 的例子：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"  
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
             xmlns:context="http://www.springframework.org/schema/context"  
             xmlns:aop="http://www.springframework.org/schema/aop"  
             xmlns:tx="http://www.springframework.org/schema/tx"  
             xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
    http://www.springframework.org/schema/context  
                   http://www.springframework.org/schema/context/spring-context-3.0.xsd  
    http://www.springframework.org/schema/aop  
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd  
    http://www.springframework.org/schema/tx  
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">  

 <bean id="student" class="com.spring.bean.Student">    
   <property name="name" value="zhangsan" />    
 </bean>    

 <bean id="school" class="com.spring.bean.School">    
 </bean>   

 <bean id="factoryBeanPojo" class="com.spring.bean.FactoryBeanPojo">    
    <property name="type" value="student" />  
 </bean>   
</beans>
```

FactoryBean 的实现类：

```java
import org.springframework.beans.factory.FactoryBean;  

/**  
 * @author  作者 wangbiao 
 * @parameter  
 * @return  
 */  
public class FactoryBeanPojo implements FactoryBean{  
    private String type;  

    @Override  
    public Object getObject() throws Exception {  
        if("student".equals(type)){  
            return new Student();             
        }else{  
            return new School();  
        }  
    }  

    @Override  
    public Class getObjectType() {  
        return School.class;  
    }  

    @Override  
    public boolean isSingleton() {  
        return true;  
    }  

    public String getType() {  
        return type;  
    }  

    public void setType(String type) {  
        this.type = type;  
    }  
}
```

普通业务类：

```java
/**  
 * @author  作者 wangbiao 
 * @parameter  
 * @return  
 */  
public class School {  
    private String schoolName;  
    private String address;  
    private int studentNumber;  
    public String getSchoolName() {  
        return schoolName;  
    }  
    public void setSchoolName(String schoolName) {  
        this.schoolName = schoolName;  
    }  
    public String getAddress() {  
        return address;  
    }  
    public void setAddress(String address) {  
        this.address = address;  
    }  
    public int getStudentNumber() {  
        return studentNumber;  
    }  
    public void setStudentNumber(int studentNumber) {  
        this.studentNumber = studentNumber;  
    }  
    @Override  
    public String toString() {  
        return "School [schoolName=" + schoolName 
                + ", address=" + address  
                + ", studentNumber=" + studentNumber + "]";  
    }  
}
```

测试类：

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;  

import com.spring.bean.FactoryBeanPojo;  

/**  
 * @author  作者 wangbiao 
 * @parameter  
 * @return  
 */  
public class FactoryBeanTest {  
    public static void main(String[] args){  
        String url = "com/spring/config/BeanConfig.xml";  
        ClassPathXmlApplicationContext cpxa = 
            new ClassPathXmlApplicationContext(url);  
        Object school = cpxa.getBean("factoryBeanPojo");  
        FactoryBeanPojo factoryBeanPojo = 
            (FactoryBeanPojo) cpxa.getBean("&factoryBeanPojo");  
        System.out.println(school.getClass().getName());  
        System.out.println(factoryBeanPojo.getClass().getName());  
    }  
}
```

输出结果：

```shell
十一月 16, 2016 10:28:24 上午 org.springframework.context.support.AbstractApplicationContext prepareRefresh  
INFO: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@1e8ee5c0: startup date [Wed Nov 16 10:28:24 CST 2016]; root of context hierarchy  
十一月 16, 2016 10:28:24 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions  
INFO: Loading XML bean definitions from class path resource [com/spring/config/BeanConfig.xml]  
十一月 16, 2016 10:28:24 上午 org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons  
INFO: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@35b793ee: defining beans [student,school,factoryBeanPojo]; root of factory hierarchy  
com.spring.bean.Student  
com.spring.bean.FactoryBeanPojo
```

从结果上可以看到当从 IOC 容器中获取FactoryBeanPojo对象的时候，用getBean(String BeanName)获取的确是Student对象，可以看到在FactoryBeanPojo中的type属性设置为student的时候，会在getObject()方法中返回Student对象。

所以说从IOC容器获取实现了FactoryBean的实现类时，返回的却是实现类中的getObject方法返回的对象，要想获取FactoryBean的实现类，得在getBean(String BeanName)中的BeanName之前加上&,写成getBean(String &BeanName)。