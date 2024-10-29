# Java Creational 创建型模式之 Factory Method 工厂方法模式
- 为了解决简单工厂模式违法开闭原则的问题，引入工厂方法模式
- 通常用一个抽象工厂类抽象产品对象的创建过程，它定义了用于创建产品对象的接口方法，然后用具体工厂子类去实例化一些产品类，工厂方法让类的实例化延迟到其工厂子类，使得客户端只需要通过指定具体工厂，而无需关心产品对象的创建细节，不需要知道这些产品类具体的类名

## 示例
```java
//产品接口
public interface IOperation {
    int operate(int a, int b);
}
//若干个实现类，比如加减乘除
public class OperationAdd implements IOperation {
    @Override
    public int operate(int a, int b) {
        return a + b;
    }
}
//抽象工厂类
public abstract class OperationFactory {
    //抽象的创建方法
    public abstract IOperation create();
}
//若干个具体工厂类
public class OperationAddFactory extends OperationFactory {
    @Override
    public IOperation create() {
        //控制对象的创建逻辑
        return new OperationAdd();
    }
}
//使用
OperationFactory operationFactory = new OperationAddFactory();
//客户端只需关心产品的接口使用，无需关心产品类具体的创建细节
IOperation operation = operationFactory.create();
operation.operate(1, 2);
```


## 特点
- 一个工厂只生产一种产品
- 将对象的创建逻辑封装在具体工厂类中，使得代码的职责更加明确，易于维护和扩展
- 减少了客户端与具体类之间的耦合，客户端无需知道具体的产品创建细节
- 当需要引入新的产品类型时，需要增加相应的具体产品对象实现类和具体工厂类，解决了简单工厂模式违法开闭原则的问题，但是这可能会导致系统中类的数量过多

