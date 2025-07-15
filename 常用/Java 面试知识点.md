# Java 面试知识点

## 面向对象的 3 大基本特性
- 封装：把客观事物封装成抽象的类，隐藏对象内部具体的实现细节，只暴露必要的方法给外部访问
- 继承：一个类可以继承另一个类的属性和方法，使得子类具有父类相同的特征和行为，子类可以重写或扩展父类的功能
- 多态：多态允许不同类对象可以用相同的接口进行交互，不同子类型的对象对同一行为可以作出不同的响应

## 面向对象的 6 大基本原则
- SRP 单一职责原则：一个类只负责一件事
- OCP 开放封闭原则（开闭原则）：对扩展开放，对修改封闭
- LSP 里氏替换原则：子类可以替换父类并且不影响程序逻辑的正确性
- ISP 接口隔离原则：应该创建多个粒度小的接口，而不是搞一个大而全的接口
- DIP 依赖倒置原则：高层模块不应该依赖于低层模块，二者都应该依赖于抽象，而抽象不应该依赖于具体实现，具体实现应该依赖于抽象
- LOD 迪米特法则（LKP 最少知识原则）

## 多线程的 3 大特性
- 可见性：当一个线程修改了该共享变量的值后，其他线程能够立刻读到修改后的值
- 原子性：指一个操作或者一系列操作是整体不可分割的，要么全部执行，要么全部不执行
- 有序性：为了让机器指令能更符合 CPU 的执行特性，在程序编译成机器码指令时可能适当的会出现指令重排的现象
- 可以使用 volatile 关键字来保证可见性和有序性，可以使用 Atomic 原子类型来保证可见性和原子性

## JMM 内存模型
- Java Memory Model 定义了 Java 程序中多线程并发访问共享内存时的原则，规定了多线程中变量的访问和修改行为，规范程序在不同的平台上都能达到内存访问的一致性，解决了并发编程的可见性、有序性两大核心问题

## 内存区域
- 虚拟机栈：存储方法调用栈帧（包括局部变量、方法返回地址等数据），支持 Java 方法的调用执行
- 本地方法栈：支持 Native 本地方法的调用执行
- 程序计数器：支持字节码指令的执行，记录当前程序的执行位置
- 堆：存储引用数据类型对象本身（包括数组）、全局变量和字符串常量池等数据，分新生代（又细分为伊甸园区和 S0 和 S1 两个 Survivor 幸存者区）和老年代
- 方法区（​元空间）：存储类元数据信息、运行时常量池等
- 局部变量：基本数据类型存储在栈中，引用数据类型对象本身存储在堆中，而引用数据类型对象的引用是存储在栈中
- 全局变量中的实例变量：存储在堆中（类对象可能包含基本数据类型和引用数据类型作为一个整体存储在堆中），每个对象都有自己独立的实例变量副本
- 全局变量中的类变量（静态变量）：存储在堆中（JDK 8 以前存储在方法区中，而 JDK 8 及以后存储在堆中的），所有对象共享同一个类变量副本

## 工厂模式
- 简单工厂：一个工厂可以生产多种产品（违反了开闭原则）
- 工厂方法：一个工厂只生产一种产品（会存在工厂子类太多的问题）
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

## IOC 控制反转（Inversion of Control）
- 将对象的创建和依赖关系的管理转交给外部容器或框架（比如 IOC 容器）去处理，而不是由对象自身来控制
- 控制反转的实现方式之一是 DI 依赖注入（Dependency Injection）
- DI 依赖注入的实现方式
    - Constructor Injection 构造器（构造函数）注入
    - Field Injection 属性（字段）注入（Setter Injection 注入）
    - Interface Injection 接口注入
    - 使用依赖注入框架（比如 Spring 框架）
- 自动依赖注入：基于 Java 反射技术实现或者在编译时自动生成连接依赖项的代码（比如 Dagger）

## Guard Clauses 卫语句
- 卫语句通常用于检查先决条件，用于在函数或方法中提前返回，以避免执行不必要的代码，这样可以提高代码的可读性和可维护性，同时减少不必要的计算或错误情况的发生
- 也可以理解为代码中的保卫者，起到检查边界，保卫代码的作用，因为这些语句在逻辑上类似于守卫，它们站在关键操作之前进行检查，确保安全和正确性

## ClassName
```java
public class AAA {
 
    class BBB {
    }
 
    public static void main(String[] args) {
        // 数组
        System.out.println(String[].class.getName()); // [Ljava.lang.String;
        System.out.println(String[].class.getCanonicalName()); // java.lang.String[]
        System.out.println(String[].class.getSimpleName()); // String[]
        System.out.println(String[].class.getTypeName()); // java.lang.String[]
 
        // 成员内部类
        System.out.println(BBB.class.getName()); // lang.reflect.AAA$BBB
        System.out.println(BBB.class.getCanonicalName()); // lang.reflect.AAA.BBB
        System.out.println(BBB.class.getSimpleName()); // BBB
        System.out.println(BBB.class.getTypeName()); // lang.reflect.AAA$BBB
 
        // 匿名内部类
        System.out.println(new Object(){}.getClass().getName()); // lang.reflect.AAA$1
        System.out.println(new Object(){}.getClass().getCanonicalName()); // null
        System.out.println(new Object(){}.getClass().getSimpleName()); // ""
        System.out.println(new Object(){}.getClass().getTypeName()); // lang.reflect.AAA$4
 
        // 普通类
        System.out.println(AAA.class.getName()); // lang.reflect.AAA
        System.out.println(AAA.class.getCanonicalName()); // lang.reflect.AAA
        System.out.println(AAA.class.getSimpleName()); // AAA
        System.out.println(AAA.class.getTypeName()); // lang.reflect.AAA
 
        // 基本数据类型
        System.out.println(int.class.getName()); // int
        System.out.println(int.class.getCanonicalName()); // int
        System.out.println(int.class.getSimpleName()); // int
        System.out.println(int.class.getTypeName()); // int
    }
}
```

## Queue 队列
- FIFO 先进先出，应当要尽量避免把 null 添加到队列里
- 通过 add/offer 方法将元素添加到队尾，通过 remove/poll 从队首获取元素并删除，通过 element/peek 从队首获取元素但不删除，每对方法的前者都会在失败时抛出异常
- Android MessageQueue 消息队列是一个单向链表实现的队列

## Deque 双端队列（Double Ended Queue）
- Deque 是一个接口，继承自 Queue，具有队列和栈的性质，队列两端的元素可以弹出
- LinkedList 是基于双向链表实现的双端队列，非线程安全，可以在任意位置高效地插入和删除元素，时间复杂度为 O(1)，访问指定位置的元素时性能较差，时间复杂度为 O(n)
- ArrayDeque 是基于动态数组实现的双端队列，可以在两端高效地插入和删除元素（可以作为栈来使用），随机访问元素时具有较好的性能，时间复杂度为 O(1)

## Stack 栈
- 又称堆栈（其实就是栈，容易误导），栈是 LIFO（Last In First Out）后进先出的线性数据结构，不推荐直接使用 java.util.Stack 类（继承自 Vector，性能较低），而是使用 java.util.Deque 接口来实现
- 只允许在栈顶进行插入和删除操，通过 push 把元素压入栈，通过 pop 把栈顶的元素弹出，通过 peek 把栈顶的元素取出但不弹出
- 应用场景：函数调用栈（保存函数上下文、局部变量、函数参数和返回地址等）、浏览器后退功能等
```java
//实现栈
Deque<Integer> stack = new ArrayDeque<Integer>();
//offerFirst 直接调用 addFirst 返回 true
//offerLast 直接调用 addLast 返回 true
//add 和 offerLast 方法一致
//pollFirst、pollLast
//removeFirst、removeLast
```

## Heap 堆
- 一种特殊的完全二叉树，通常用数组存储实现
- 最大堆（大根堆、大顶堆）：父节点的值大于等于所有子节点的值，根节点是最大值
- 最小堆（小根堆、小顶堆）：父节点的值小于等于所有子节点的值，根节点是最小值
- 通过 offer 插入元素（up 上浮操作），通过 poll 删除根元素（down 下沉操作），通过 heapify 堆化
```java
//PriorityQueue 优先队列，其逻辑结构是一棵完全二叉树，而其存储结构其实是一个数组
PriorityQueue<Integer> minHeap = new PriorityQueue<>(); //默认是最小堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a); //通过自定义比较器实现最大堆
```
## Tree 树
- 二叉树：每个节点最多有两个子节点
- Binary Search Tree 二叉查找树（Ordered Binary Tree 有序二叉树、Sorted Binary Tree 排序二叉树）：是一种特殊的二叉树，其中每个节点的左子树所有节点的值都小于该节点的值，右子树所有节点的值都大于该节点的值
- 平衡树：为了解决二叉查找树在频繁插入和删除操作后可能出现的不平衡问题（在极端情况下可能会退化为链表的结构），从而引入了平衡树（将二叉查找树平衡均匀地分布，可以有效减少二叉查找树的深度），比如 Red-Black Tree 红黑树，通过颜色规则​​和​​旋转操作保持平衡以确保操作的时间复杂度（保证查找、插入和删除操作的时间复杂度均为 O(log n)），从而提高了性能
- 完全二叉树：假设一个二叉树其深度为 d（d>1），除了第 d 层以外其它各层的节点数均已达最大值，且第 d 层所有节点从左向右连续地紧密排列
- 满二叉树：所有叶节点都在最底层的完全二叉树

## List 列表
- ArrayList、Vector、CopyOnWriteArrayList 和 Collections$SynchronizedList
- ArrayList 非线程安全，效率较高，底层数据结构采用数组实现的，初始大小为 10
- Vector 线程安全，效率较低，底层原理和 ArrayList 基本一致，两者均继承自 AbstractList，ArrayList 和 Vector 的方法基本都是一样的，只不过 Vector 的方法大多都添加了 synchronized 方法锁，虽然能够保证线程安全，但性能就降低了（不推荐使用）
- CopyOnWriteArrayList 写时复制一份新数组进行写操作，读操作在原数组上进行，因此读操作无需加锁，适用于读多写少的场景
- Collections$SynchronizedList 可以将一个普通的 List 转换为线程安全的 List，性能略优于 Vector，适用于读写操作相对均衡的场景
- PS：LinkedList 底层数据结构为双向循环链表，相对于 ArrayList 来说插入和删除的操作效率较高，查找和修改的操作效率较低

## Map 映射
- HashMap、ConcurrentHashMap
- HashMap 基于哈希表的 Map 接口实现，用来存放 key-value 键值对
- LinkedHashMap 保留插入顺序，适合需要保持输入顺序的场景

## ArrayList 和 Array 数组之间互转
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


## 线程池
- Executors#newCachedThreadPool 缓存线程池，只有非核心线程，空闲时间超过 60s 就会被回收
- Executors#newFixedThreadPool 固定大小的线程池，只有核心线程，空闲也一直存在不会被回收

核心线程数未满：如果当前运行的线程数少于核心线程数（corePoolSize），线程池会创建一个新线程来执行新任务。
核心线程数已满，但任务队列未满：如果当前运行的线程数已达到核心线程数，但任务队列（workQueue）尚未满，线程池会将新任务加入任务队列，等待有空闲线程时执行。
任务队列已满，且最大线程数未满：如果任务队列已满，但当前线程数还未达到最大线程数（maximumPoolSize），线程池会创建新线程来执行新任务。
任务队列已满，且已达到最大线程数：如果任务队列已满，且当前线程数已达到最大线程数，线程池会根据配置的拒绝策略（handler）来处理新任务，如直接抛出异常、丢弃任务等。

BlockingQueue
直接提交队列（SynchronousQueue）：此队列实际上不存储任何元素，它直接将每个插入操作必须等待一个对应的移除操作，反之亦然。它通常用于创建无界线程池（如newCachedThreadPool），在这种情况下，最大线程数实际上取决于系统（或JVM）的限制。
无界队列（如LinkedBlockingQueue）：此队列按照先进先出（FIFO）的排序规则对元素进行排序，且队列的容量为Integer.MAX_VALUE。使用无界队列时，线程池的大小将只受到corePoolSize的约束，因为任务可以在队列中无限期地等待执行。然而，这可能导致资源耗尽，特别是当每个任务都长时间运行时。
有界队列（如ArrayBlockingQueue）：此队列是一个基于数组结构的有界阻塞队列，新元素插入队列时，如果队列已满，则插入线程会被阻塞直到队列中有空间可用。使用有界队列时，可以更好地控制资源的使用，防止资源耗尽。
优先级队列（如PriorityBlockingQueue）：此队列根据元素的优先级进行排序，优先级高的元素先被移除。这对于需要按优先级执行任务的应用场景非常有用。


## 时间复杂度
在表示算法的执行时间时，通常会使用大 O 表示法，常见的标识类型有以下这些
O(1)：常量时间，计算时间与数据量大小没关系；
O(n)：计算时间与数据量成线性正比关系；
O(logn)：计算时间与数据量成对数关系；

## ReentrantLock
ReentrantLock 底层是使用 AQS 队列 + CAS 操作来实现的


