# Flutter 知识点
- Flutter 是一个由 Google 开发的开源 UI 工具包，用于构建跨平台移动应用、Web 应用、桌面应用和嵌入式应用，它使用的是 Dart 编程语言，具有快速的热重载功能和丰富的现代化 UI 组件，使得开发者能够轻松构建高性能、美观的应用程序

https://docs.flutter.cn/get-started/flutter-for/android-devs

## Flutter 特点
- Hot Reload 热重载
- Flutter 中一切皆是 Widget 呈现的，大致对应 Android 中的 View（不完全对应），不会直接绘制任何内容，而是 UI 及其底层创建真正视图对象的语义的描述
- 在 Flutter 中，Widget 是不可变的，无法被直接更新，需要操作 Widget 的状态
- 底层渲染引擎 Skia，使用自己的渲染引擎来绘制 UI，能够保证高性能，媲美原生应用的速度   利用 Skia 绘制，实现跨平台应用层渲染一致性
Skia：Flutter 使用 Skia 作为其渲染引擎。Skia 是一个高性能的 2D 图形库，可以直接与底层操作系统的图形 API 进行交互（如 OpenGL 和 Vulkan），从而实现高效的图形渲染
自绘 Widget：Flutter 的 UI 是完全由 Widgets 构成的，Flutter 不依赖于原生 UI 组件，而是通过绘制其自己的组件来实现。从而确保了在不同平台上具有一致的外观和行为

Flutter 提供热重载功能，使得开发者在进行 UI 修改时可以立即查看效果，而无需重新启动应用，这大大提高了开发效率。

在 Flutter 中，通过组合更小的 Widget 来创建自定义 Widget（而不是继承它们，这和 Android 中实现一个自定义的 ViewGroup 有些类似）
暂不支持从 Flutter 中直接调用 Native 代码

Flutter 使用 Ahead-of-Time (AOT) 编译，将 Dart 代码编译为原生机器码，从而提高应用的启动速度和运行性能


## Flutter 官方 package
https://pub.dev/packages?q=publisher%3Aflutter.dev&sort=like

##  Dart 官方 package
https://pub.dev/packages?q=publisher%3Adart.dev&sort=like


mixin是一个定义类的关键字。直译出来是混入，混合的意思 Dart为了支持多重继承， 引入了mixin关键字，它最大的特殊处在于： mixin定义的类不能有构造方法，这样可以避 免继承多个类而产生的父类构造方法冲突
# Dart
Dart没有public protected private等关键字  下划线 _ 开头直接代表是私有的


在Dart中，一切都是对象，所有的对象都是继承自Object
dart是值传递。我们每次调用函数，传递过去的都是对象的内存地址，而不是这个对象的复制
Dart是强类型语言，但可以用var或 dynamic来声明一个变量，Dart会自动推断其数据类型
 Dart 属于是强类型语言 ，但可以用 var 来声明变 量，Dart 会自推导出数据类型，var 实际上是编译期 的“语法糖”。dynamic 表示动态类型， 被编译后， 实际是一个 object 类型，在编译期间不进行任何的 类型检查，而是在运行期进行类型检查
Dart  所有异步的操作的返回值都用 Future 来表示
Dart 中统一使用 Stream 处理异步事件流
dart 是强引用
## Widget、Element 和 RenderObject
- Widget 用来描述配置信息，Element 负责差异比对，RenderObject 负责实际渲染
- Widget 是不可变的，因此配置改变时需要重新构建 UI

## StatelessWidget 和 StatefulWidget
. 什么是StatelessWidget和StatefullWidget？他们之间的区别是什么？
StatelessWidget是一旦构建后状态就不能改变的Widget，无生命周期的回调。
StatefulWidget是一旦构建Widget的状态还会发生改变的Widget。
StatefulWidget有一个状态类State，维护了可变状态。当状态发生变化时，StatefulWidget 将会重建其对应的 State 对象。
生命周期不同，调用build()方法次数和时机不同。

## 解决项目移动后编译报错的一种方式
删除 pubspec.lock 文件
执行 flutter pub get 命令

## http

## Dio



flutter的channel通信有哪几种？你用的哪种？插件你如何实现的？



使用 isolate 的具体场景: 可能需要几百毫秒的
1、JSON解析: 解码JSON，这是HttpRequest的结果，可能需要一些时间，可以使用封装好的 isolate 的 compute 顶层方法。
2、加解密: 加解密过程比较耗时
3、图片处理: 比如裁剪图片比较耗时
4、从网络中加载大图 



Flutter 是如何与原生Android、iOS进行通信的？
Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：

BasicMessageChannel ：用于传递字符串和半结构化的信息。
MethodChannel ：用于传递方法调用（method invocation）。
EventChannel : 用于数据流（event streams）的通信。
 


6. 什么是Flutter中的异步编程？有哪些常用的异步编程模式？
Flutter中的异步是向事件队列中插入任务。
Future、Stream
Future一次只支持一个任务，Stream可以支持多个任务。
async和await、await for是异步的语法糖。


Flutter 是如何实现原生性能和体验的？
Flutter 通过一系列独特的设计和技术实现了原生应用的性能和体验。以下是 Flutter 如何实现原生的几个关键点：

渲染引擎

Skia：Flutter 使用 Skia 作为其渲染引擎。Skia 是一个高性能的 2D 图形库，可以直接与底层操作系统的图形 API 进行交互（如 OpenGL 和 Vulkan），从而实现高效的图形渲染。
直接访问原生 API

Platform Channels：Flutter 通过平台通道（Platform Channels）与原生代码进行通信。开发者可以在 Flutter 中调用原生 Android 或 iOS 的 API，实现对硬件功能（如相机、GPS 等）的访问。
Widget 树

自绘 Widget：Flutter 的 UI 是完全由 Widgets 构成的，Flutter 不依赖于原生 UI 组件，而是通过绘制其自己的组件来实现。从而确保了在不同平台上具有一致的外观和行为。
高效的性能

AOT 编译：Flutter 使用 Ahead-of-Time (AOT) 编译，将 Dart 代码编译为原生机器码，从而提高应用的启动速度和运行性能。
热重载

开发体验：Flutter 提供热重载功能，使得开发者在进行 UI 修改时可以立即查看效果，而无需重新启动应用，这大大提高了开发效率。
跨平台

单一代码库：通过共享单一代码库，Flutter 可以同时为 iOS 和 Android 平台构建应用，减少了开发和维护的成本。
丰富的组件库

Material 和 Cupertino：Flutter 提供了丰富的 Material Design 和 Cupertino 组件，开发者可以轻松创建符合 Android 和 iOS 平台的原生用户体验。



JIT 和 AOT

Flutter 中的 JIT（Just-in-Time）和 AOT（Ahead-of-Time）是两种不同的编译方式，用于将 Flutter 代码转换成可执行的机器代码。



## 为什么要将 build 方法放在 State 中，而不是放在 StatefulWidget 中
- 提高开发的灵活性
- 每次状态改变都要调用build方法，如果放在 StatefulWidget 中就很不方便了


Provider是继承于InheritProvider，而InheritProvider本质上是一个InheritWidget，所以Provider本质上是依托于InheritProvider的机制来实现的widget树的状态共享



