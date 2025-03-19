# Android 面试知识点

## 四大组件
- App Component 应用组件包括 Activity、Service、ContentProvider 内容提供者和 BroadcastReceiver 广播接收器，应用组件可以不按顺序地单独启动，并且系统或者用户可以随时销毁它们，因此不应该在应用组件中存储任何应用数据或状态，并且应用组件不应该相互依赖

## Activity 生命周期
```java
onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```

## Service 生命周期
```java
//通过 startService 方法启动
onCreate —— onStartCommand —— onDestroy
//通过 bindService 方法绑定
onCreate —— onBind —— onUnbind —— onDestroy
```

## Activity launchMode 启动模式
- 1 standard 标准模式（默认）
- 2 singleTop 栈顶复用模式（比如：通知栏推送点击消息界面），如果复用就走 onNewIntent 回调
- 3 singleTask 栈内复用模式（比如：应用的首页），如果复用就走 onNewIntent 回调，同时清除其之上的所有 Activity 实例
- 4 singleInstance 单例模式（单独位于一个任务栈中，比如：拨打电话界面、呼叫来电界面、浏览器）

## Activity、Window 和 DecorView
- Activity 的创建：ActivityThread#handleLaunchActivity -> ActivityThread#performLaunchActivity -> Instrumentation#newActivity
- Window 的创建：ActivityThread#handleLaunchActivity -> ActivityThread#performLaunchActivity -> Activity#attach
- DecorView 的创建：Activity#setContentView -> PhoneWindow#setContentView -> PhoneWindow#installDecor -> PhoneWindow#generateDecor

```java
//Activity 和 Window 都是抽象概念的，是肉眼看不见的，而实际看得见其实就是 DecorView
Activity 负责承载 UI 用户界面、处理用户事件交互和管理生命周期
-Window（PhoneWindow）是一种抽象概念，用于描述一个窗口，负责确定窗口外观属性（比如窗口大小、位置、背景、标题、颜色、透明度、全屏和分屏等）、控制窗口行为策略
--DecorView（FrameLayout）就是对 FrameLayout 进行了功能的修饰，是顶层布局容器，用于实际承载视图的，它通常包含了状态栏、标题栏和内容区域等
---ContentRoot（ViewGroup，跟据主题样式决定，比如 ActionBarOverlayLayout、LinearLayout 等）
----ActionBar（标题栏）
----ContentParent（FrameLayout，id 是 android:id/content）
-----ContentView（setContentView 设置 layoutResID）
```

## Activity 初始化流程
- 关键方法 ActivityThread#performLaunchActivity 主要干了三件事：1 创建 Activity 实例，2 初始化 Context 和 Application， 3 执行 Activity 生命周期方法
- Instrumentation 仪表的作用：1 负责 Activity 实例的创建，2 对 Activity 生命周期进行监控和控制-----------监控程序和系统之间的交互的Instrumentation 类主要用来监控应用程序与系统交互

```java
Activity#startActivity //启动一个 Activity
-Activity#startActivityForResult
--Instrumentation#execStartActivity //与 ActivityManagerService 进行跨进程通信
---ActivityTaskManager.getService().startActivity //通过 Binder 跨进程通信请求 AMS，然后经过一系列操作后，再由 AMS 通过 Binder 跨进程通信调用触发 ActivityThread 进行处理启动目标 Activity，最终会调用到 ActivityThread#handleLaunchActivity 方法
----ActivityThread#handleLaunchActivity 
```

```java
ActivityThread#handleLaunchActivity //处理 Activity 启动
-ActivityThread#performLaunchActivity //执行完成 Activity 启动,内部进行 Activity 的 Context 的初始化，紧接着进行 Application 的初始化，
--Instrumentation#newActivity //内部进行 Activity 的初始化，Instrumentation#newActivity 利用 ClassLoader 类加载器通过反射创建对应的 Activity 实例
--Activity#attach //内部进行 Window（PhoneWindow）的初始化，Window#setWindowManager
--Instrumentation#callActiviyOnCreate
---Activity#performCreate
----Activity#onCreate //最终会调用 Activity#onCreate 方法
```

```java
ActivityThread#handleResumeActivity
-ActivityThread#performResumeActivity
--Activity#performResume
---Instrumentation#callActivityOnResume
----Activity#onResume //最终会调用 Activity#onResume 方法
```

## Window 初始化流程
- PhoneWindow
```java
ActivityThread#handleLaunchActivity
-ActivityThread#performLaunchActivity
--Instrumentation#newActivity //内部进行 Activity 的初始化
--Activity#attach //内部进行 Window（PhoneWindow）的初始化
--Instrumentation#callActiviyOnCreate
```

## DecorView 初始化流程
- 涉及 Activity#setContentView 的原理
- 如果多次调用 Activity#setContentView 的话，新布局会替换之前的布局，会重复解析 xml 和构建 View 树
```java
Activity#setContentView //设置 layoutResID，内部就是调用了 PhoneWindow#setContentView 方法
-PhoneWindow#setContentView
--PhoneWindow#installDecor
---PhoneWindow#generateDecor //进行 DecorView（FrameLayout）的初始化
---DecorView#setWindow(phoneWindow)
---PhoneWindow#generateLayout //根据主题样式确定一个 xml 文件作为 DecorView 的子布局 ContentRoot，默认是 screen_simple.xml（对应是 LinearLayout）
----DecorView#onResourcesLoaded //把 xml 构建成 View 树（利用 LayoutInflater#inflate 通过 LoadXmlResourceParser 进行解析）
```
## ActivityManagerService 的作用
- 负责四大组件的启动、切换、调度
- 应用程序进程的管理、调度

## Window、WindowManager 和 WindowManagerService
- Window 包含并管理 View，WindowManager 管理 Window，而 WindowManager 的操作是通过和 WindowManagerService 交互来实现的

## WindowManager、WindowManagerImpl 和 WindowManagerGlobal
- WindowManager 是一个接口，继承自 ViewManager 接口，WindowManagerImpl 是 WindowManager 的接口实现类
- WindowManager 的作用是对 Window 进行管理，包括增加、更新和删除等操作（定义在 ViewManager 接口里）
- WindowManagerImpl 方法内部又是调用 WindowManagerGlobal 的相关方法，涉及到桥接模式的知识

## App 启动流程
- Launcher App -- Binder --> ActivityManagerService（AMS 在 SystemServer 进程中）
```java
Launcher#startActivitySafely //带上 FLAG_ACTIVITY_NEW_TASK 标记让 Activity 启动在新的任务栈中
-Activity#startActivity
--Activity#startActivityForResult
---Instrumentation#execStartActivity
----ActivityManager.getService().startActivity //通过 Binder 跨进程通信请求 AMS，然后会调用到 ActivityManagerService#startActivity 方法
```

```java
ActivityManagerService#startActivity
-ActivityManagerService#startActivityAsUser
--ActivityStarter#startActivityUnchecked
---ActivityStarter#startActivityInner
----ActivityTaskSupervisor#startSpecificActivity //判断 App 的应用程序进程是否已经启动
```

应用程序进程已启动
- ActivityManagerService -- Binder --> ActivityThread（主线程管理类）
```java
ActivityTaskSupervisor#startSpecificActivity
ActivityTaskSupervisor#realStartActivityLocked //ActivityThread 启动目标 Activity
-ActivityThread$H#sendMessage //通过 Handler 调用 H#sendMessage 方式发送通知
--ActivityThread#handleLaunchActivity //处理根 Activity 启动
---ActivityThread#performLaunchActivity
----Instrumentation#newActivity //Activity 的初始化
----Activity#attach
----Instrumentation#callActivityOnCreate
-----Activity#performCreate
------Activity#onCreate
```

应用程序进程未启动
- ActivityManagerService -- Socket --> Zygote
```java
//startSpecificActivity 和 realStartActivityLocked 中增加额外逻辑
ActivityTaskSupervisor#startSpecificActivity
-ActivityTaskManagerService#startProcessAsync
--ActivityManagerService#startProcessLocked //通过 Socket 通知 Zygote 进程孵化创建（Zygote 会启动 Socket 服务来监听等待，利用 fork 自身方式创建）应用程序进程
---ZygoteInit#main
---ActivityThread#main //进程孵化创建后会调用 ActivityThread#main 方法
---ActivityThread#attach
----ActivityThread.H#handleMessage
-----Instrumentation#newApplication
-----Instrumentation#callApplicationOnCreate
----ActivityTaskSupervisor#realStartActivityLocked
```


## View 绘制流程
- 关键方法 ViewRootImpl#scheduleTraversals 会触发 ViewRootImpl#performTraversals 去执行测量、布局和绘制三大方法
```java
ActivityThread#handleResumeActivity
-ActivityThread#performResumeActivity
-WindowManager.addView(decorView) //就是 ViewManager#addView，WindowManager 接口继承了 ViewManager 接口，作用就是添加 DecorView 到 Window
--WindowManagerImpl#addView //WindowManagerImpl 实现了 WindowManager 接口
```

```java
WindowManagerImpl#addView
-WindowManagerGlobal#addView //内部进行 ViewRootImpl 的初始化
--ViewRootImpl#setView //通过 setView 方法设置关联的 DecorView（保存了 DecorView 实例），然后再通过 DecorView.assignParent 把 ViewRootImpl 设置成 DecorView 的 ViewParent（ViewRootImpl 不是一个真正的 View）
---ViewRootImpl#requestLayout //触发了第一次 View 的布局绘制
----ViewRootImpl#scheduleTraversals //内部有 Handler 同步屏障逻辑 MessageQueue#postSyncBarrier
-----Choreographer#postCallback //设置传入一个 Choreographer.CALLBACK_TRAVERSAL 类型和一个 mTraversalRunnable 对象，Runnable#run 方法里就是执行 ViewRootImpl#doTraversal 方法
------ViewRootImpl#doTraversal
-------ViewRootImpl#performTraversals //内部代码逻辑就是依次按照条件判断是否去执行 performMeasure、performLayout 和 performDraw 方法
--------ViewRootImpl#performMeasure
--------ViewRootImpl#performLayout
--------ViewRootImpl#performDraw
```


## ViewRootImpl
- ViewRootImpl 的创建：ActivityThread#handleResumeActivity -> WindowManagerImpl#addView -> WindowManagerGlobal#addView
- ViewRootImpl 是 WindowManager 和 DecorView 之间的桥梁，用来管理 View 的各种事件，包括 invalidate、requestLayout、dispatchInputEvent 等
- ViewRootImpl 是 DecorView 的 ViewParent，也正因为这样 View#requestLayout 层层调用最终能调到 ViewRootImpl#requestLayout

## 卡顿优化
1、 利用UI线程的Looper打印的日志匹配；  是blockcanary的原理，就是利用looper.loop分发事件的时间间隔作为卡顿的依据
当我们进行invalidate或者requestLayout时，总会执行viewRootImp的scheduleTraversals方法

2、 使用Choreographer.FrameCallback
也就是对logging进行深入的研究，一般超过了1000ms就认为卡顿了，但我们有没有想过，我们通常说的卡顿不是说超过了16.66ms么，为何这里要超过500ms,甚至1000ms才算是卡顿？
我们要知道，android系统所有的执行都是基于looper机制的，也就是所有的消息执行的时间超过1000ms就认定卡顿了，举个例子，我们可能在主线程操作数据库，可能在主线程解析json，可能在主线程写文件，可能在主线程做一些例如高斯模糊的耗时操作
这种情况下我们利用blockcanary是可以检测出来的，但是如果是卡顿呢？当然我们也可以把时间设定为50ms，但是检测出来的太多了，所以就需要第二个机制了Choreographer.FrameCallback,通常这样写
可以看到这里是CALLBACK_ANIMATION的callback，也就说只要监听了此方法他就会不断的调用doFrame，
在doframe里调用postFrameCallback，从而来达到完美的监听ui卡顿的效果
也就是onMeasure,onLayout,onDraw的耗时时间


## onNewIntent
setIntent(intent); 更新 intent


## Context

## Fragment
- Fragment 允许将界面分成为好几个区块，从而将模块化和可重用性能力引入 Activity 
- 支持同一功能界面根据屏幕大小不同可以实现两个版本的页面显示样式

## ContentProvider
- ContentProvider 通常会在 Application#onCreate 方法之前初始化，多个 ContentProvider 的初始化顺序遵循 AndroidManifest.xml 文件中的声明顺序

## App Startup
- 很多三方库（比如 LeakCanary）都用 ContentProvider 的小技巧进行 Library 的初始化操作，初始化很多 ContentProvider 一定程度上会影响性能，官方就出了 App Startup 统一到一个 ContentProvider 里去初始化，提供了一个 ContentProvider 来运行所有依赖项的初始化
- 利用 ContentProvider 实现初始化 Library 获取 Context

## Binder IPC 进程间通信机制
- Binder 是 Android 提供的一种 IPC 进程间通信机制，Binder 是一种基于消息传递的 IPC 机制，基于 C/S 架构
- AIDL 是用来抽象化 Binder IPC 的工具（接口的定义）
- 不需要两次拷贝，利用内存映射，只进行了一次拷贝

## Handler 消息机制
- Handler 消息机制是一套设计用来进行线程间通信的机制，主要是用于实现线程切换，能够让子线程间接的去访问 UI 控件，在 Java 层及 Native 层均是由 Handler、Looper、MessageQueue 三者构成

### Handler
- 消息处理器，负责收发消息，Handler 发 Message 消息到 MessageQueue 消息队列里，而消息队列中的 Message 中存放着的 Handler 引用，在 dispatchMessage 分发消息的时会将消息发到该 Handler 里进行处理
 
### Looper
- 一个 Handler 对应一个 Looper，每个线程只能有一个 Looper

### MessageQueue
- 用于存储 Message，内部维护了 Message 的链表，每次拿取 Message 时，若该 Message 离真正执行还需要一段时间，会通过 nativePollOnce 进入阻塞状态，避免资源的浪费，若存在消息屏障，则会忽略同步消息优先拿取异步消息，从而实现异步消息的优先消费

### 异步消息
- 在创建 Handler 时，Handler 构造函数有一个 async 参数，默认传 false，若 async 传 true 则表示这个 Handler 发出的消息均为异步消息，异步消息会被优先处理

### 同步屏障
- 同步屏障的作用就是为了保证异步消息可以优先执行
- 可以通过 MessageQueue#postSyncBarrier 方法来设置同步屏障，该方法发送了一个没有 target（Handler）的 Message，在 MessageQueue#next 方法中获取消息时，如果发现没有 target 的 Message，则在一定的时间内跳过同步消息，优先执行异步消息。再换句话说，同步屏障为 Handler 消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息
- 子线程中创建 Handler

在创建Handler时有一个async参数，传true表示此handler发送的时异步消息。ViewRootImpl.scheduleTraversals方法就使用了同步屏障，保证UI绘制优先执行

### 消息机制的原理
- ActivityThread#main 方法中通过 Looper#prepareMainLooper 调用 Looper#prepare 创建一个 Looper , 涉及 ThreadLocal 存取 Looper 对象，而 Looper  的构造方法里创建了一个 MessageQueue
- ActivityThread#main 方法继续执行，里面有个 Looper.loop 方法，内部利用 while 循环在主线程中开启了一个死循环，用它不断得从 MessageQueue 消息队列里取 Message（如果没有消息就利用 Linux epoll 机制进行阻塞等待），然后分发消息给 target（Handler）处理


- Handler post 方法原理
- Handler.postDelayed ：修改手机系统时间对延迟消息不会有影响
 
## HandlerThread
- 是一个自带 Looper 的线程

## IdleHandler 空闲处理器
- 通常用于在消息队列空闲的时候去执行一些低优先级、轻量级的任务，可以实现一些延迟初始化的应用
- IdleHandler 是一个回调接口，可以通过MessageQueue的addIdleHandler添加实现类。当MessageQueue中的任务暂时处理完了（没有新任务或者下一个任务延时在之后），这个时候会回调这个接口，返回false，那么就会移除它，返回true就会在下次message处理完了的时候继续回调

Handler机制。MessageQueue中的Message是如何排列的？Msg的runnable对象可以外部设置么，比如说不用Handler#post系列方法（反射可以实现）；

## SyncBarrier 同步屏障
- 同步屏障的特点是 msg.target = null，就是没有关联 Handler 的消息，相当于一个标志位，通过 MessageQueue#postSyncBarrier 发送，通过 MessageQueue#removeSyncBarrier 进行移除
- ViewRootImpl#scheduleTraversals 方法就是通过  mHandler.getLooper().getQueue().postSyncBarrier()  添加同步屏障以确保来保证 UI 绘制优先执行

## Init、Zygote 和 SystemServer 进程
- Init 由 Linux 系统内核启动，解析 init.rc 文件后 fork 生成 Zygote 进程
- Zygote 由 Init 进程启动，是所有 App 应用程序进程的父进程
- SystemServer 负责启动各种系统核心服务

## IntentService
- 单线程执行：Service 默认在主线程中运行，而 IntentService 是运行在一个子线程中，不会阻塞主线程，默认无需去手动管理线程
- 自动停止：IntentService 不需要手动停止

## JobScheduler

##  !!!!!!后台任务解决方案
- 如果是一个长时间的 HTTP 下载的话就使用 DownloadManager
- 否则的话就看是不是一个可以延迟的任务，如果不可以延迟就直接使用 Foreground Service
- 如果可以延迟的话就看是不是可以由系统条件触发，如果是的话就使用 WorkManager
- 如果不是就看是不是需要在一个固定的时间执行这个任务，如果是的话就使用 AlarmManager
- 如果不是的话就还是使用 WorkManager

## 后台 Service 和子线程的区别
后台 Service
- Service 是四大组件之一，自身不提供 UI 元素
- 默认是运行在主线程的，耗时操作需要开子线程进行操作（可以选用 IntentService）
- 可以不依赖 Activity 而存在，能做到程序关闭后仍旧能继续执行，能够长时间运行
- 后台的概念主要是它不和 UI 打交道，是运行在后台的服务，最多通知前台 UI 更新
子线程
- 对应主线程的说法
- 后台的概念主要是能够异步运行

## Lifecycle
- Lifecycle 基于观察者模式实现的
- LifecycleOwner 接口
- LifecycleObserver 接口
- 利用 LifecycleRegistry 对象来管理生命周期状态

## LiveData
- LiveData 是一个可观察的数据持有类，旨在存储和传递数据，遵循观察者模式，基于 Lifecycle 生命周期感知组件进行工作
- SafeIterableMap 是一个基于双向链表的数据结构
- 粘性事件：LiveData 默认被设计为 Sticky Events 粘性事件（事件发送后，观察者才订阅，能收到订阅之前的事件）的
- 数据倒灌：可以理解成在粘性事件前提条件的基础上，当第二次调用 observe 时候，如果还能收到第一次调用 observe 旧的观察者已经处理过的数据的情形

## ViewModel
- ViewModel 实现在 Activity 和 Fragment 之间共享数据，可以在配置更改（比如屏幕旋转）后仍旧能保持数据，从而避免了数据丢失和重复初始化
- ViewModelStoreOwner 接口
- 每个 Activity 或 Fragment 都有自己对应的 ViewModelStore，通过唯一 Key 值将 ViewModel 实例存储到 ViewModelStore 中

## View invalidate 和 postInvalidate 的区别
- postInvalidate 可以用在非 UI 主线程中，postInvalidate 方法最后调用的也是 invalidate 方法
- 任意一个子 View 执行 View#invalidate 最终会调用 ViewRootImpl#scheduleTraversals 调度遍历接着 doTraversal 然后 performTraversals 执行遍历，下来就是 performMeasure、performLayout（因为条件不成立会跳过这两个方法）和 performDraw

## View invalidate 和 requestLayout 的区别
- View#requestLayout 最终也会调用到 ViewRootImpl#performTraversals 调度遍历接着 doTraversal 然后 performTraversals 执行遍历，下来就是 performMeasure、performLayout 和 performDraw
- 只修改比如颜色、文本等用 invalidate
- 只修改尺寸、位置等用 requestLayout
- 可以先调用 requestLayout 来更新布局，然后调用 invalidate 来触发重绘

## View 绘制机制
- 由 ViewRootImpl 来统一协调管理整个视图树的绘制流程，处理来自系统的输入事件分发和生命周期变化
- 每个 Activity 的 Window 都对应一个 ViewRootImpl 实例（将 ViewRootImpl 和 DecorView 建立关联）

## View 生命周期
- 在 Activity 的 onResume 方法里是无法获取 View 正确的宽高的
- 官方推荐采用 onWindowFocusChanged(true) 回调来确定当前 View 所在的 Activity 是对用户可见的并且可交互的，所以这里可以获取到 View 正确的宽高
- onAttachedToWindow 进行资源准备初始化，onDetachedFromWindow 进行资源释放取消等

## 动画
- Frame 逐帧动画是通过按顺序快速播放一系列图像（通常称为帧）来形成动画效果的技术
- Tween 补间动画不会改变视图的实际属性，只是改变了视图在视觉上呈现动画效果，所以不适合用在有事件交互的场景
- Property 属性动画能对对象的任意属性（比如位置、大小、颜色、透明度等）进行动画处理，如果需要对自定义属性实现动画效果需要保证自定义属性对应的 getter 和 setter 方法是可被访问的

## 命令式编程和响应式编程的区别
- 命令式编程强调步骤和过程，侧重于 “怎么做”
- 响应式编程强调以声明式的方式处理异步数据流，关注于定义数据流及其变换规则，而不是控制执行流程，更侧重于“做什么”

## 内存泄漏
- 内存泄漏：本该被 GC 回收的内存实际没有被回收
- 根本原因：短生命周期的对象被长生命周期的对象持有引用，导致短生命周期对象的内存在其生命周期结束后无法被 GC 回收，从而导致了内存泄露
- 非静态内部类或匿名内部类都会隐式地持有外部类的引用，都可以访问外部类的变量
- 减少单例、静态对象、非静态内部类或匿名内部类的使用

## LeakCanary
- LeakCanary 使用 弱引用 来跟踪 Activity，并结合 GC 机制判断 Activity 是否被回收，从而识别潜在的内存泄漏问题

## 事件分发机制
- MotionEvent 事件产生后，按照 Activity ->  Window -> DectorView -> View 顺序传递的过程就叫事件分发
- View#dispatchTouchEvent 分发事件
- ViewGroup#onInterceptTouchEvent 拦截事件（只有 ViewGroup 有）
- View#onTouchEvent 处理事件

OnTouchListener -> OnTouchEvent -> OnClick

## MediaCodec
- MediaCodec 类可以访问底层的编解码器，用于编解码音视频。它提供了一种直接处理压缩数据（例如AAC, MP3, H.264等）的方法。
- MediaCodec 编码摄像头画面 MediaMuxer 合并生成 MP4 文件
- MediaCodec 解码播放 MediaExtractor 提取 MP4 文件的内容
- AudioTrack 播放音轨


使用 MediaCodec，你可以进行诸如视频播放、视频编辑、实时通信等操作。要使用 MediaCodec 进行解码，通常需要先配置媒体格式，然后调用 configure() 方法，接着是 start()，之后可以通过 dequeueInputBuffer() 和 queueInputBuffer() 来输入数据，并通过 dequeueOutputBuffer() 获取输出的数据


## 编码格式和容器格式
- 编码格式： H.264/AVC H.265/HEVC MP3 AAC
- 容器格式： MP4 MKV 

## HTTP 超文本传输协议
- HTTP 是一种以明文形式传输数据，简单、灵活可扩展的，基于 TCP/IP 协议传输数据，可靠的传输协议
- HTTPS 超文本传输安全协议，通过增加 SSL/TLS 协议来加密 HTTP 数据传输
- HTTP/2 作为 HTTP 协议的第 2 个主要版本，引入了二进制分帧、多路复用、头部压缩、服务器推送、请求优先级等特性

## 缓存机制
三级缓存：内存、硬盘、网络
LRU：Least Recently Used 最近最少使用，常用于页面置换算法、常用作缓存淘汰策略

Android 缓存机制
> 狭义上是指针对网络的缓存
>
> 广义上指的是对数据的复用

 
 LRU如何实现  如何自己实现一个LRUCache？Android里面的LRUCache是如何实现的


## 缓存策略

- FIFO
- LFU Least Frequently Used 最不经常使用
- LRU


## LruCache



## RecyclerView 缓存机制
- ViewHolder 主要解决了 ItemView 复用的问题，减少重复 findViewById 的消耗
四级缓存：
    - 屏幕内缓存：mAttachedScrap（数据未改变）和 mChangedScrap（数据已改变）临时保存屏幕内的 ViewHolder
    - 屏幕外缓存：mCachedViews 最近被移出屏幕的缓存，默认缓存数量是 2 个，本质上是一个 ArrayList，当 mCachedViews 满了之后，新移除的 ViewHolder 会被放入 RecycledViewPool 中
    - 自定义缓存：ViewCacheExtension
    - 共享缓存池： RecycledViewPool，默认缓存数量是 5 个，本质上是一个 SparseArray，其缓存的 ViewHolder 是全新的，复用时需要重新绑定数据（重新调用 bindViewHolder）


## DiffUtil


## Alibaba ARouter
- 用于实现 Android 组件化开发的路由框架，它通过实现不同模块之间的页面跳转和参数传递等功能，使得不同的业务模块之间进行解耦，可以更加独立地开发和维护各个模块
- 可以使 Android 应用的架构更加清晰、可维护性更高
- 支持降级处理，如果目标页面或服务不可用，可以进行降级处理，提高应用的稳定性
- 拥有控制拦截能力，可以配置拦截器来控制页面跳转