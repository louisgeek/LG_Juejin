# Java 策略模式和状态模式的区别
- 策略模式和状态模式都属于行为型模式，两者在结构上很相似，但在设计意图和应用场景存在显著差异
- 策略模式将一系列算法（策略）中的每一个算法独立封装成类，算法之间允许相互替换，客户端通过统一接口调用不同策略（在运行时动态选择），实现算法的定义与使用分离
- 状态模式将对象的状态和行为独立封装成类，状态的变化由对象自身管理，对象内部封装了状态转换逻辑，使对象在不同状态下表现出不同行为（状态变化时自动切换行为）

## 策略模式
- 实现算法（策略）的动态替换，新增策略无需修改现有代码，适用于支付方式、排序算法和折扣策略等场景
- Strategy：抽象策略类，定义算法接口、公共方法
- ConcreteStrategy：具体策略类，实现了抽象策略类，提供具体的算法实现
- Context：上下文类，持有一个策略对象，通过接口方法调用算法
```java
public interface DiscountStrategy {
    double applyDiscount(double price);
}
public class NormalDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(double price) { 
        return price * 0.95; 
    }
}
public class VIPDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(double price) { 
        return price * 0.85; 
    }
}
//上下文类
public class ShoppingCart {
    //持有一个策略对象
    private DiscountStrategy strategy;
    //通过构造函数设置
    public ShoppingCart(DiscountStrategy strategy) { 
        this.strategy = strategy; 
    }
    //通过 Setter 设置
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    public double checkout(double price) {
        //通过接口方法调用算法
        return strategy.applyDiscount(price); 
    }
}
public class Client {
    public static void main(String[] args) {
        //
        ShoppingCart shoppingCart = new ShoppingCart(new VIPDiscount());
        shoppingCart.checkout(100);
        //
        shoppingCart.setStrategy(new NormalDiscount());
        shoppingCart.checkout(50);
    }
}
```

## 状态模式
- 实现状态改变时直接触发行为变化，状态对象行为可以随状态连续变化，适用于订单状态、红绿灯状态等场景
- State：抽象状态类，定义状态接口、状态相关行为
- ConcreteState：具体状态类，实现了抽象状态类，内部包含状态转移逻辑
- Context：上下文类，持有一个当前状态对象
```java
public interface OrderState {
    void handle(OrderContext context);
}
//待支付状态
public class PendingState implements OrderState {
    @Override
    public void handle(OrderContext context) {
        System.out.println("订单待支付，触发支付提醒");
        context.setState(new PaidState()); //支付后切换成已支付状态
    }
}
//已支付状态
public class PaidState implements OrderState {
    @Override
    public void handle(OrderContext context) {
        System.out.println("订单已支付，准备发货");
        context.setState(new ShippedState()); //发货后切换成发货状态
    }
}
//已发货状态
public class ShippedState implements OrderState {
    @Override
    public void handle(OrderContext context) {
        System.out.println("订单已发货，等待用户确认");
        context.setState(null); //最终状态无后续
    }
}
//上下文类
public class OrderContext {
    //持有一个当前状态类对象
    private OrderState currentState;
    //通过构造函数设置
    public OrderContext(OrderState state) { 
        this.currentState = state; 
    }
    //通过 Setter 设置
    public void setState(OrderState state) { 
        this.currentState = state; 
    }
    public void processOrder() {
       if (currentState != null) {
           currentState.handle(this);
       } else {
            System.out.println("订单已完成");
       }
    }
}
public class Client {
    public static void main(String[] args) {
        OrderContext context = new OrderContext(new PendingState());
        context.processOrder(); //订单待支付，触发支付提醒
        context.processOrder(); //订单已支付，准备发货
        context.processOrder(); //订单已发货，等待用户确认
    }
}
```

## 总结
- 策略模式强调算法的独立性和互换性，状态模式强调状态驱动的行为变化和状态转换逻辑的封装
- 策略类不需要持有 Context，而状态类通常需要持有 Context，用于状态切换
- 策略模式由客户端主动选择策略，而状态模式由状态类自身管理驱动状态转换
- 在复杂系统中可结合两种模式混合使用​​，比如使用策略模式选择支付方式，使用状态模式管理支付流程