# Java Design Patterns 设计模式
- GoF 在《设计模式》这本书里列举并描述了 23 种设计模式

## Creational Patterns 创建型模式
- Singleton 单例（比如 Calendar#getInstance、InputMethodManager#getInstance）
- Factory Method 工厂方法
- Abstract Factory 抽象工厂（比如 BaseActivity）
- Builder 建造者（比如 AlertDialog.Builder）
- Prototype 原型

  额外有个 Simple Factory 简单工厂模式

## Structural Patterns 结构型模式
- Adapter 适配器（比如 ListView 或 RecyclerView 的 Adapter）
- Bridge 桥接
- Proxy 代理（比如 AppCompatActivity 的 AppCompatDelegate）
- Decorator 装饰器
- Composite 组合
- Facade 外观
- Flyweight 享元（比如 Message#obtain）

## Behavioral Patterns 行为型模式
- Observer 观察者（比如 LiveData、ContentObserver）
- Chain of Responsibility 责任链
- Strategy 策略
- Template Method 模板方法
- Iterator 迭代器
- Command 命令（比如 Handler#post）
- Memento 备忘录
- State 状态
- Visitor 访问者
- Mediator 中介者
- Interpreter 解释器

 额外有个 Pipeline 管道模式，是责任链模式的常用变种之一


## 简单工厂模式、工厂方法模式和抽象工厂模式的区别
- 简单工厂模式：通过工厂类创建对象，并且根据传入参数决定具体子类对象的做法就是简单工厂模式，通常会用到 if else，新增或这删除子类都要修改工厂类的代码，违反了开闭原则
- 工厂方法模式：为了解决简单工厂模式存在的违反开闭原则的问题，给每个子类都对应一个工厂子类，利用多态特性创建对象的模式，就是工厂方法模式，然而为每个子类都创建一个工厂子类就显得有点太繁琐了
- 抽象工厂模式：为了解决工厂方法模式的工厂子类太多的问题，可以采用抽象工厂模式，把不同的子类分个组，每组使用同一个工厂类的不同方法来创建


## 适配器模式和桥接模式的区别
 - 都是结构型设计模式，通常都可以用来解决接口不兼容问题的方法

### 适配器模式
- 将一个类的接口转换成客户所期望的另一个接口，使得原本由于接口不兼容而无法一起工作的类能够协同工作
- 主要用于解决两个已有接口间的匹配问题，其中被适配的接口的实现往往是不能修改的
- 常见的应用场景就是出国用的插头适配器，国标和海外标准的转换器

### 桥接模式
- 将抽象部分与它具体实现的部分进行分离，使它们都可以独立地变化
- 适用于当一个类存在两个独立变化的维度，且这两个维度都需要进行扩展的情况
- 可以避免由于多层继承导致的类爆炸问题


## 代理模式和装饰器模式的区别
- 都是结构型设计模式，通常都可以使用组合的方式实现，即一个类持有另一个类的引用，并通过这个引用来调用被包装对象的方法
- 都涉及到一个对象包装另一个对象以控制对被包装对象的访问或扩展其功能，可以在不修改被包装对象的基础上，通过包装对象来增加额外的功能或控制对被包装对象的访问

### 代理模式
- 通过引入代理类来给原始对象添加一些额外的功能或控制访问权限，关注于控制访问和优化性能，如权限检查控制、缓存优化、延迟加载和日志记录等

### 装饰器模式
- 通过创建装饰器类来实现，这些装饰器类包装原始对象，并在调用原始对象的方法前后添加额外的功能逻辑，关注于扩展功能，如图像旋转缩放、图像模糊、视频滤镜和文本加粗等
- 通过将功能划分成小的装饰器类，以灵活的方式组合这些装饰器类来增强原始对象的功能，而无需直接修改原始对象的代码
- Android RecyclerView 的 addItemDecoration 添加分割线的相关方法就是装饰器模式的应用
