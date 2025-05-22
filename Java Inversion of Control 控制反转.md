# Java Inversion of Control 控制反转
- 简称 IOC，控制反转是一种设计原则（理念），核心思想是将对象的创建和依赖关系的管理转交给外部容器或框架（比如 IOC 容器）去处理，而不是由对象自身来控制，减少程序之间的耦合，提高系统的灵活性、可测试性和可维护性
- 控制反转的实现方式
    - Dependency Injection 依赖注入
    - Factory Pattern 工厂模式
    - Dependency Lookup 依赖查找
    - Service Locator Pattern 服务定位器模式
    - Event-driven 事件驱动
- 控制：指控制对象的创建和销毁，即控制对象的生命周期
- 反转：指依赖对象的获得方式被反转了，以前是客户端直接 new 出依赖项的实例，现在则是转移交给了 IOC 容器根据配置（比如 Xml 文件、注解等）去创建和控制
- Java 中实现控制反转主要综合使用了工厂模式、Xml 解析、Java 反射等技术

## Dependency Injection 依赖注入
- 简称 DI，依赖注入是实现控制反转的最常用方式
- 依赖注入的实现方式
    - Constructor Injection 构造器（构造函数）注入
    - Field Injection 属性（字段、参数）注入（Setter Injection 注入）
    - Interface Injection 接口注入
    - 使用依赖注入框架（比如 Spring 框架）
 
### 手动依赖注入
- 1 最原始的方式就是在使用的地方按顺序实例化依赖项然后注入，意味着业务逻辑处出现了大量样板代码
- 2 自行创建实现一个依赖项容器类，比如叫 AppContainer，由于各个地方都有可能使用到它，所以通常可以写在 Application 类里，但这意味着必须自行管理 AppContainer，手动为所有依赖项创建实例，仍存在部分样板代码

### 自动依赖注入
- 1 基于 Java 反射技术的解决方案，可以在运行时连接依赖项（比如 Guice，不过可能存在性能问题）
- 2 静态解决方案，可在编译时自动生成连接依赖项的代码，（比如 Google 的 Dagger 或者 Hilt，不过 Dagger 和 Hilt 可以共存）

## 控制反转和依赖倒置原则
- 依赖倒置原则是 Java 面相对象设计基本原则之一，强调高层模块和低层模块都应该依赖于抽象，而不是具体的实现
- 依赖倒置原则为控制反转提供了理论基础，控制反转是依赖倒置原则在实践中的一种有效实现方式