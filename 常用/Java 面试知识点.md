# Java 面试知识点

## 面向对象的 3 大基本特性
- 封装：把客观事物封装成抽象的类，隐藏对象内部具体的实现细节，只暴露必要的方法给外部访问
- 继承：一个类可以继承另一个类的属性和方法，使得子类具有父类相同的特征和行为，子类可以重写或扩展父类的功能
- 多态：多态允许不同类对象可以用相同的接口进行交互，不同子类型的对象对同一行为可以作出不同的响应

## 面向对象的 6 大基本原则
- SRP 单一职责原则
- OCP 开放封闭原则，开闭原则，对扩展开放，对修改封闭
- LSP 里氏替换原则，子类可以替换父类，不影响程序逻辑正确性
- ISP 接口隔离原则
- DIP 依赖倒置原则
- LOD 迪米特法则，最少知识原则

## 多线程的 3 大特性
- 可见性：当一个线程修改了该共享变量的值后，其他线程能够立刻读到修改后的值
- 原子性：指一个操作或者一系列操作是整体不可分割的，要么全部执行，要么全部不执行
- 有序性：为了让机器指令能更符合 CPU 的执行特性，在程序编译成机器码指令时可能适当的会出现指令重排的现象
- 可以使用 volatile 关键字来保证可见性和有序性，可以使用 Atomic 原子类型来保证可见性和原子性
 
## 工厂模式
- 简单工厂：一个工厂可以生产多种产品，违法开闭原则
- 工厂方法：一个工厂只生产一种产品，存在工厂子类太多的问题
- 抽象工厂：工厂和产品之间是多对多的关系

## 观察者模式
- 被观察者会维护一个观察者列表，当被观察者的状态发生改变时会遍历观察者列表依次通知每个观察者

## 责任链模式
- 允许将请求沿着一条若干个处理者的链上进行传递，每个处理者都有机会处理请求，直到请求被处理或到达链的末端
- 侧重于条件判断决定某一个处理者进行处理，审批流程、事件处理

## 管道模式
- 管道模式也称为流水线模式，是责任链模式的常用变种之一，通常情况下一个步骤会依赖上一个步骤输出的结果
- 侧重处理者都处理一部分事情，日志处理、图像视频处理

## 代理模式
- 在不改变目标类的情况下通过代理类增加功能来扩展目标类的逻辑功能，真正的业务逻辑还是由目标类来实现

## Inversion of Control 控制反转
- 将对象的创建和依赖关系的管理转交给外部容器或框架（比如 IOC 容器）去处理，而不是由对象自身来控制
- 控制反转的实现方式之一是 DI 依赖注入
- DI 依赖注入的实现方式
    - Constructor Injection 构造器（构造函数）注入
    - Field Injection 属性（字段）注入（Setter Injection 注入）
    - Interface Injection 接口注入
    - 使用依赖注入框架（比如 Spring 框架）
- 自动依赖注入：基于 Java 反射技术实现或者在编译时自动生成连接依赖项的代码（Dagger）

## Queue 队列
- FIFO 先进先出，应当要尽量避免把 null 添加到队列里
- 通过 add / offer 方法将元素添加到队尾，通过 remove / poll 从队首获取元素并删除，通过 element  / peek 从队首获取元素但不删除，每对方法的前者都会在失败时抛出异常

?LinkedList 是双向链表实现的 Deque 双端队列 
?Android MessageQueue 消息队列，它是一个单向链表实现的队列

## Stack 堆栈
- LIFO 后进先出，不推荐直接使用 Stack 类（继承自 Vector，性能较低），而是改用 Deque 接口
- 通过 push 把元素压入栈，通过 pop 把栈顶的元素弹出，通过 peek 把栈顶的元素取出但不弹出
```java
Deque<Integer> stack = new ArrayDeque<Integer>();
```

## List
- ArrayList、Vector、CopyOnWriteArrayList 和 Collections$SynchronizedList
- ArrayList 非线程安全，效率较高
- Vector 线程安全，效率较低，不推荐使用
- CopyOnWriteArrayList 写时复制一份新数组进行写操作，读操作在原数组上进行，因此读操作无需加锁，适用于读多写少的场景
- Collections$SynchronizedList 可以将一个普通的 List 转换为线程安全的 List，性能略优于 Vector,适用于读写操作相对均衡的场景


Vector 底层原理和 ArrayList 基本一致，底层数据结构都是采用数组（Object[] elementData 数组存储数据）实现的，两者均继承自 AbstractList，ArrayList 和 Vector 的方法基本都是一样的，只不过 Vector 的方法大多都添加了 synchronized 方法锁

LinkedList 底层数据结构为双向循环链表，非线程安全的，相当于 ArrayList 来说插入和删除的操作效率较高，查找和修改的操作效率较低


## Map
- HashMap、ConcurrentHashMap



# ArrayList 和 Array 数组之间互转
- java.util.ArrayList -> Array 数组
```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
//List 直接转成数组
String[] array = list.toArray(new String[list.size()]);
```
- Array 数组 -> java.util.ArrayList 
```java
String[] array = new String[] {"1", "2", "3","4"};
//注意这里返回的是 java.util.Arrays.ArrayList 而不是 java.util.ArrayList
//可以调 set、get 等方法，但不能调 add、remove 等方法，因为没有实现，调用会报 UnsupportedOperationException 异常
List<String> listTemp = Arrays.asList(array);
List<String> list = new ArrayList<>(listTemp);
```


