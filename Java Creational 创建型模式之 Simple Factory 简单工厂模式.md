# Java Creational 创建型模式之 Simple Factory 简单工厂模式
- 也叫静态工厂方法模式
- 将一些产品对象的创建（比如 new）与使用进行分离的方法，客户端不需要知道这些产品类具体的类名，通常用一个单独的工厂类去实例化这些产品类，通过传入参数（比如 type 类型）去返回对应产品类的实例对象，一般这些产品类都有共同的接口或者父类（那如果没有呢？那不就是类似 Helper 类了，此时传入参数概念就类比成调用具体的方法了）

```java
SimpleFactory.create("A")
SimpleFactory.create("B")
 
//Helper 类
SimpleHelper.buildA()
SimpleHelper.buildB()
```

## 示例
```java
//产品接口
public interface IOperation {
    int operate(int a, int b);
}
//若干个产品实现类，比如加减乘除
public class OperationAdd implements IOperation {
    @Override
    public int operate(int a, int b) {
        return a + b;
    }
}
//工厂类
public class OperationFactory {
    //静态方法统一管理和控制对象的创建逻辑
    public static IOperation create(String operation) {
        IOperation iOperation = null;
        switch (operation) {
            case "+":
                iOperation = new OperationAdd();
                break;
            case "-":
                iOperation = new OperationMinus();
                break;
            case "*":
                iOperation = new OperationMul();
                break;
            case "/":
                iOperation = new OperationDiv();
                break;
        }
        return iOperation;
    }
}
//使用
//客户端只需关心产品的接口使用，无需关心产品类具体的创建细节
IOperation operation = OperationFactory.create("+");
operation.operate(1, 2);
```


## 特点
- 一个工厂可以生产多种产品
- 减少了客户端与具体类之间的耦合，同时代码简单易懂，易于实现
- 将产品对象的创建逻辑集中在一个单独的类中，利于统一管理和控制，代码的维护和修改方便，不会直接影响到客户端的代码，但是当需要引入新的产品类型时，需要修改工厂类的代码，这违反了开闭原则
- 工厂类的职责过重，它既要负责创建产品对象，又要知道所有产品类的具体实现逻辑，可能会导致工厂类变得庞大和复杂
- java.util.Calendar 就是简单工厂模式的应用，Calendar#getInstance 静态方法就是通过传入时区和语言环境返回对应类的实例对象