# Java Creational 创建型模式之 Abstract Factory 抽象工厂模式
- 一定程度上为了解决工厂方法模式的工厂子类太多的问题，可以利用抽象工厂模式，把一系列相关的不同的子类分个组，每组使用同一个工厂类的不同方法来创建对象
- 通常用一个抽象工厂类定义若干个接口，多个具体工厂类实现这些接口，以及定义一系列抽象产品和具体产品类，多个抽象产品组成的产品组

## 示例
```java
//产品接口
public interface IProductA {
}
public interface IProductB {
}
//若干个产品实现类
public class ProductA1 implements IProductA {

}
public class ProductA2 implements IProductA {

}
public class ProductB1 implements IProductB {

}
public class ProductB2 implements IProductB {

}
//抽象工厂类
public interface ProductFactory {
    IProductA createProductA();

    IProductB createProductB();
}
//若干个具体工厂类
public class ProductFactory1 implements ProductFactory {

    @Override
    public IProductA createProductA() {
        return new ProductA1();
    }

    @Override
    public IProductB createProductB() {
        return new ProductB1();
    }
}
public class ProductFactory2 implements ProductFactory {

    @Override
    public IProductA createProductA() {
        return new ProductA2();
    }

    @Override
    public IProductB createProductB() {
        return new ProductB2();
    }
}
//使用
  ProductFactory productFactory1 = new ProductFactory1();
  IProductA productA1  =  productFactory1.createProductA();
  IProductB productB1 = productFactory1.createProductB();  

  ProductFactory productFactory2 = new ProductFactory2();
  IProductA productA2 =  productFactory2.createProductA();
  IProductB productB2 = productFactory2.createProductB();
```


## 特点
- 工厂和产品之间是多对多的关系，不同工厂可以生产同类的产品，一个产品可以交给不同工厂去生产
- 将对象的创建逻辑封装在具体工厂类中，使得代码的职责更加明确，易于维护和扩展
- 减少了客户端与具体类之间的耦合，客户端无需知道具体的产品创建细节
- 在需要创建一系列相关产品时更加方便和高效，不过当需要引入新的产品类型时，所有的工厂类都需要进行修改，可能增加了系统的复杂性和维护成本