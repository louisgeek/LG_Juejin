# Java Behavioral 行为型模式之 Observer 观察者模式
- 观察者模式定义了对象之间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知并自动更新，其中发生改变的对象称为 Observable 被观察者（又叫 Subject 主题），接收通知并进行相应处理的对象称为 Observer 观察者
- 被观察者会维护一个观察者列表，并提供添加、移除和通知观察者的通用方法，当被观察者的状态发生改变时，会遍历观察者列表依次通知每个观察者
- 被观察者和观察者之间可以通过抽象接口进行交互，他们可以独立地进行修改和扩展，而不需要相互了解对方的具体实现
- 观察者模式允许随时添加新的观察者或删除现有的观察者，这种灵活性提高了系统的可扩展性，使得系统能够适应不断变化的需求，符合开闭原则

被观察者
```java
public interface Observable {
    void addObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}
//
public class MyObservable implements Observable {
    //维护一个观察者列表
    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
        //当状态改变时，通知所有观察者
        this.notifyObservers();
    }

    @Override
    public void notifyObservers() {
        //通知所有观察者
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}
```

观察者
```java
public interface Observer {
    void update(String message);
}
//
public class MyObserver implements Observer {
    //
    private final String name;

    public MyObserver(String name) {
        this.name = name;
    }

    @Override
    public void update(String message) {
        //处理消息
        System.out.println(name + " 收到消息: " + message);
    }
}
```

使用
```java
public static void main(String[] args) {
    //创建被观察者
    MyObservable myObservable = new MyObservable();
    //创建若干个观察者
    Observer observer1 = new MyObserver("观察者1");
    Observer observer2 = new MyObserver("观察者2");
    Observer observer3 = new MyObserver("观察者3");
    //添加若干个观察者
    myObservable.addObserver(observer1);
    myObservable.addObserver(observer2);
    myObservable.addObserver(observer3);
    //数据变化
    myObservable.setMessage("测试消息1");
    //移除一个观察者
    myObservable.removeObserver(observer1);
    //数据变化
    myObservable.setMessage("测试消息2");
    //-------- 结果 --------
    //观察者1 收到消息: 测试消息1
    //观察者2 收到消息: 测试消息1
    //观察者3 收到消息: 测试消息1
    //观察者2 收到消息: 测试消息2
    //观察者3 收到消息: 测试消息2
}
```

## 应用场景
- 1 事件监听：用户的操作（如点击按钮、输入文本等）可以触发事件，这些事件可以被多个监听器监听并进行相应的处理
- 2 消息推送：构建消息通知系统时订阅者视作观察者，而发布者视作被观察者，当发布者发布新消息时，所有订阅者都能够接收到通知
- 3 消息队列：消息队列系统通常使用观察者模式来实现的消息传递，生产者将消息发送到队列中，而消费者作为观察者订阅特定类型的消息会在变化的时候得到通知并去处理

## 特点
- 1 降低耦合度：观察者模式将主题与具体观察者进行一定程度的解耦，使它们可以独立地变化和复用，这样降低了系统各部分的依赖程度，提高了系统的可维护性
- 2 易于扩展：观察者模式支持在运行时动态添加和删除观察者，这样得能够根据实际需求灵活地进行调整，使得应用程序可以很容易地扩展
- 3 不必要的更新：如果观察者对发布者的某些状态变化不感兴趣，但仍然收到了通知进行不必要的更新操作，可能导致不必要的更新
- 4 性能问题：在有大量观察者的情况下，频繁的状态变化触发所有观察者的更新可能会导致性能下降，可能引发性能问题，可以考虑有策略的进行通知
- 5 常见应用：Android 的 Lifecycle 生命周期感知组件和 LiveData 组件


