# Flutter 面试知识点

## Flutter 简介
- 跨平台：共享同一份代码，跨平台 2D 绘图引擎
- 高性能：自绘引擎 Skia，Dart 支持 AOT 提前编译
- 热重载：实时查看代码修改效果，提高开发效率
- 灵活的 UI 设计理念：采用 “一切皆是 Widget” 的设计理念，声明式 UI
- Dart 类型安全和空安全：Dart 是强类型的语言，支持编译期检查

## Dart 简介
- Dart 中，一切都是对象，所有对象都继承自 Object（包括数字、null）
- Dart 是单继承的
- Dart 支持顶层函数，函数是一等公民
- Dart 是值传递（Java 也是值传递）：当一个参数按照值的方式在两个方法之间传递时，调用者和被调用者其实是用的两个不同的变量（被调用者中的变量是调用者中变量的一份拷贝，任一变量修改都不会影响到另一个变量）
- Dart 使用 _ 下划线开头表示私有属性，没有 public 和 private 等关键字
- Dart 是单线程模型

## Dart final 和 const 的区别
- final 和 const 都是用于声明不可变的变量的关键字
- final 和 const 声明的变量在赋值后都是不可变的（只能赋值一次，不能被重新赋值），但 const 的限制更加严格（相当于 const 包含了 final 的特性，所以 final 和 const 不能一起使用）
- final 变量的值可以在运行时确定，而 const 变量的值必须在编译时就能计算出来（不能在运行时动态计算）
- final 强调引用的不可变性，const 强调值和内存的不可变性

## Dart Future 和 Stream 的区别
- Future 用于表示一个尚未完成的异步操作的结果，承诺最终会返回一个值，Stream 用于表示一系列异步事件的序列（数据流），可以持续发送多个数据
- Future 无需手动关闭，Stream 需要手动关闭
- Future 的所有 API 的返回值仍然是一个 Future 对象，所以可以很方便的进行链式调用
- Future 适用于如网络请求、数据库查询、文件读取和延迟任务等，Stream 适用于如传感器数据、实时推送、键盘输入、轮询请求和定时器等

## Dart Object、dynamic 和 var 的区别
- Object、dynamic 和 var 是三种不同的变量声明方式，主要区别在于类型检查、类型推断和类型安全性等方面
- Object 通用类型、dynamic 动态类型和 var 类型推断
- Object 可以存储任意类型，适合泛型等应用场景
- dynamic 需要谨慎使用，仅在必要时使用，比如 JSON 解析处理未知结构的数据
- var 优先推荐使用，能够简化代码书写而且保证类型安全

## Dart Stream 的 2 种类型
- 单订阅流是默认的 Stream 类型，只能被一个监听器订阅（多次订阅会抛出异常，即使取消后也无法重复订阅）
- 单订阅流适用于一对一的异步事件处理，事件需要按顺序提供并且不能丢失（比如网络请求、文件读取等）
- 广播流允许多个监听器同时订阅，数据会广播给所有订阅者
- 广播流适用于一对多的事件广播场景，即使没有监听者，事件也仍会被发送，不再使用后需要手动关闭（比如用户登录状态变化（状态广播）、传感器数据监听等）
- 广播流的局限性：不能保留已发送的事件（新的订阅者无法收到旧的已发送的时间）

## Dart Mixin 混入
- Mixin 是一种实现代码复用的机制，允许宿主类通过 with 关键字“混入”封装好的功能模块（即复用 Mixin 的方法和属性），而不是通过传统继承或实现接口的方式，解决了 Dart 单继承的限制（可以实现类似 “多继承” 的效果），使得代码更加灵活且避免了传统多继承的复杂性
- Mixin 可以包含方法和属性，但不能被实例化，其目的就是为了提供可复用的代码，专门用于被其他类 “混入”
- Mixin 可以定义抽象方法，强制宿主类实现
- 可以通过多个 Mixin 灵活组合使用，将日志记录、网络请求、数据校验和权限校验等通用功能逻辑分别封装成一个个 Mixin，避免代码重复
- 可以使用 on 限制 Mixin 的应用范围
- 如果多个 Mixin 或父类都定义了同名方法，会按线性化顺序选择最后混入的 Mixin（最后混入的优先级最高，变量的解析规则和方法一致）

## Flutter StatelessWidget 和 StatefulWidget 的区别
- StatelessWidget 和 StatefulWidget 是构建 UI 用户界面的两种基础组件，StatelessWidget 适用于静态 UI，而 StatefulWidget 适用于需要动态更新的 UI
- StatelessWidget 只依赖于构造函数中的参数来构建 UI，不维护任何内部状态，在需要更新 UI 时，必须重新构建该 Widget 的实例，而 StatefulWidget 维护一个可变的 State 状态，可以响应用户输入或其他事件并更新 UI，当状态发生变化时，通过调用 setState 方法来通知触发 UI 更新
- StatelessWidget 无状态管理，性能较好，适合静态内容（比如 Text 文本、Icon 图标和 Container 容器等），StatefulWidget 适合需要动态更新的场景（如表单输入、列表刷新、计数器和动画等）

## Flutter Widget、Element 和 RenderObject 的区别
- Widget、Element 和 RenderObject 的三者通过分工（描述、管理和渲染）和协作，实现高效、灵活的 UI 渲染和更新机制
- Widget 用于声明式地（声明式 UI）描述界面元素的外观和行为，不直接处理渲染绘制，Element 是 Widget 树在引擎中的实例化对象，是 UI 树的真实节点，负责管理 Widget 的生命周期和状态，并创建对应的 RenderObject，RenderObject 是实际负责布局和绘制的对象，是渲染树中的节点，直接操作底层渲染引擎（比如 Skia）
- 一个 Widget 对应一个 Element，而一个 Element 不一定对应一个 RenderObject，Element 协调 Widget 与 RenderObject 之间的绑定关系（可以理解为是一个粘合剂），当 Widget 更新时，Element 会对比新旧 Widget 的差异，决定是否复用 Element，避免重复创建 Element 和 RenderObject（因为 Widget 不涉及渲染，所以更新代价很小，而 RenderObject 负责 UI 渲染，所以更新代价极大）


## Flutter PlatformChannel 平台通道
- PlatformChannel 平台通道是实现 Flutter 与原生平台之间通信的核心机制，通过三种不同类型的平台通道来实现数据传递和方法调用，分为 MethodChannel、EventChannel 和 BasicMessageChannel
- MethodChannel 是最常用的通道，用于 Flutter 端和原生端之间进行方法调用和返回结果（双向同步，请求-响应模式），适合一次性调用（比如获取设备信息）
- EventChannel 用于 Flutter 端持续接收原生端的事件流（单向异步，比如传感器数据监听、网络状态监听和电池状态监听等），EventChannel 底层也是 MethodChannel 实现的
- BasicMessageChannel 用于 Flutter 端和原生端之间进行字符串或二进制等信息的传递，适合双向通信、简单数据交换和大文件传输（编码器：StandardMessageCodec（默认）、StringCodec、JSONMessageCodec 和 BinaryCodec），双向异步
```dart
//MethodChannel
MethodChannel#invokeMethod 调用方法
MethodChannel#setMethodCallHandler

//EventChannel
EventChannel#receiveBroadcastStream#listen 进行数据监听
EventChannel#setStreamHandler
EventSink.success 发送消息

//BasicMessageChannel
BasicMessageChannel#send 发送消息
BasicMessageChannel#setMessageHandler
```



## Flutter Explicit 显式动画


## Flutter Implicit 隐式动画


## Flutter ChangeNotifier 和 ValueNotifier 的区别
- ChangeNotifier 和 ValueNotifier 是两种用于状态管理工具，两者均基于发布-订阅模式设计（采用观察者模式）
- ChangeNotifier 需要自定义状态变量，而 ValueNotifier 可以直接使用内置 value 属性存储状态
- 单值简单状态优先采用 ValueNotifier，简单直观，多值复杂状态用 ChangeNotifier，更灵活，支持复杂逻辑扩展自定义，更适合复杂状态管理
- 对于多个关联值，应该改用 ChangeNotifier，而非多个 ValueNotifier 实现


## Flutter状态管理
Provider是继承于InheritProvider，而InheritProvider本质上是一个InheritWidget，所以Provider本质上是依托于InheritProvider的机制来实现的widget树的状态共享


## isolate
art是如何实现多任务并行的
主要依赖dart的并发编程（Isolate）、异步和事件驱动机制
 Isolate Groups如何解决内存隔离问题？
   - 传统Isolate内存不共享，Isolate Groups允许同组Isolate共享**不可变数据**  
   - 通过`Isolate.spawnUri`的`shared`参数启用，适合只读数据场景

## 
如何获取控件的大小和位置？
使用Key拿到上下文取得findRenderObject拿内容的尺寸数据；
使用context取得findRenderObject拿内容的尺寸数据；


## Future、async和await
await的出现会把await之前和之后的代码分为两部分，await并不像字面意思所表示的程序运行到这里就阻塞了，而是立刻结束当前函数的执行并返回一个Future，函数内剩余代码通过调度异步执行。
async是和await搭配使用的，await只在async函数中出现。在async 函数里可以没有await或者有多个await。

 
# await for如何使用？
await for是用来不断获取stream流中的数据，然后执行循环体中的操作。它一般用在直到stream什么时候完成，并且必须等待传递完成后才能使用，不然会阻塞。
. await for 如何使用？ await for是不断获取stream流中的数据，然后执行循环体中的操作。它一般用在直到stream什么时候完成，并且必须等待传递完成之后才能使用，不然就会一直阻塞。

 Dart 当中的 「..」意思是 「级联操作符」，为了方便配置而使用。「..」和「.」不同的是 调用「..」后返回的相当于是 this，而「.」返回的则是该方法返回的值


## Widget 唯一标识Key

 ## Widget、State、Context 的核心概念


get set方法实现

简述flutter中自定义View流程

1，已有控件（widget）的继承，组合

2，自定义绘制widget,也就是利用paint，cavans等进行绘制视图。

## flutter_boost
的优缺点，内部实现



runApp()和main()有什么区别？
1main() 和runApp() 函数在flutter的作用分别是什么？有什么关系吗？

main函数是类似于java语言的程序运行入口函数

runApp函数是渲染根widget树的函数

一般情况下runApp函数会在main函数里执行

## Navigator
什么是Navigator? MaterialApp做了什么？


## Fultter 网络请求
Http
Dio



## StatefulWidget 的生命周期
- initState: 当 State 对象插入视图树中时调用，通常用于完成一些初始化工作
- didChangeDependencies: 当 State 对象的依赖关系发生变化时会被调用，例如 InheritedWidget 发生变化时
- build: 返回需要渲染的 Widget，此方法会被多次调用，因此应避免在此方法中执行耗时操作
- didUpdateWidget: 当父组件重新构建时调用，例如父组件的状态发生变化
- deactivate: 当组件从视图树中移除（但还没有被销毁）时会被调用
- dispose: 当组件被永久移除时调用，通常用于释放资源

