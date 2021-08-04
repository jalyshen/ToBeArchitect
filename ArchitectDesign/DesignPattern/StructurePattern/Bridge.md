# 桥接模式

原文：http://c.biancheng.net/view/1364.html



## 1. 定义与特点

​        桥接（Bridge）模式的定义：**将抽象与实现分离，使它们可以独立变化**。它是***用组合关系代替继承关系***来实现，从而降低了抽象和实现这两个可变维度的耦合度。

​        桥接模式遵循了里氏替换原则和依赖倒置原则，最终实现了开闭原则，对修改关闭，对扩展开放。

​        桥接模式的优点是：

* 抽象与实现分离，扩展能力强
* 符合开闭原则
* 符合合成复用原则
* 其实现细节对客户透明

​       缺点是：由于聚合关系建立在抽象层，要求开发者针对抽象化进行设计与编程，能正确的识别出系统中两个独立变化的维度，者增加了系统的理解与设计难度。

## 2. 结构与实现

​        可以将抽象化部分与实现化部分分开，取消二者的继承关系，该用组合关系。

### 2.1 模式的结构

​        桥接模式包含以下主要的角色：

1. 抽象化（Abstraction）角色：定义抽象类，并包含一个对实现化对象的引用
2. 扩展抽象化（Refined Abstraction）角色：是抽象化角色的子类，实现父类中的业务方法，并通过组合关系调用实现化角色中的业务方法
3. 实现化（Implementor）角色：定义实现化角色的接口，供扩展抽象化角色调用
4. 具体实现化（Concrete Implementor）角色：给出实现化角色接口的具体实现

结构图如下所示：

![1](../../images/StructurePattern/Bridge/1.gif)

### 2.2 模式的代码

​        桥接模式的代码：

```java
package bridge;

public class BridgeTest {
    public static void main(String[] args) {
        Implementor imple = new ConcreteImplementorA();
        Abstraction abs = new RefinedAbstraction(imple);
        abs.Operation();
    }
}

//实现化角色
interface Implementor {
    public void OperationImpl();
}

//具体实现化角色
class ConcreteImplementorA implements Implementor {
    public void OperationImpl() {
        System.out.println("具体实现化(Concrete Implementor)角色被访问");
    }
}

//抽象化角色
abstract class Abstraction {
    protected Implementor imple;

    protected Abstraction(Implementor imple) {
        this.imple = imple;
    }

    public abstract void Operation();
}

//扩展抽象化角色
class RefinedAbstraction extends Abstraction {
    protected RefinedAbstraction(Implementor imple) {
        super(imple);
    }

    public void Operation() {
        System.out.println("扩展抽象化(Refined Abstraction)角色被访问");
        imple.OperationImpl();
    }
}
```

运行结果：

```
扩展抽象化(Refined Abstraction)角色被访问
具体实现化(Concrete Implementor)角色被访问
```

## 3. 应用实例



## 4. 应用场景



## 5. 模式的扩展

