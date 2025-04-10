# Android 面试知识点

## 四大组件
- App Component 应用组件包括 Activity、Service、ContentProvider 内容提供者和 BroadcastReceiver 广播接收器，应用组件可以不按顺序地单独启动，并且系统或者用户可以随时销毁它们，因此不应该在应用组件中存储任何应用数据或状态，并且应用组件不应该相互依赖

## Activity 生命周期
- 6 个
- 锁屏不走 onStop
- 下拉状态栏不走生命周期
```java
onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```

## Fragment 生命周期
- 11 个
```java
onAttach —— onCreate —— onCreateView —— onViewCreated —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestroyView —— onDestroy —— onDetach
```

## Service 生命周期
```java
//通过 startService 方法启动
onCreate —— onStartCommand —— onDestroy
//通过 bindService 方法绑定
onCreate —— onBind —— onUnbind —— onDestroy
```

## Activity launchMode 启动模式
- standard 标准模式（默认）
- singleTop 栈顶复用模式（比如：通知栏推送点击消息界面），如果复用就走 onNewIntent 回调
- singleTask 栈内复用模式（比如：应用的首页），如果复用就走 onNewIntent 回调，同时清除其之上的所有 Activity 实例
- singleInstance 单例模式（单独位于一个任务栈中，比如：拨打电话界面、呼叫来电界面、浏览器）

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
- 关键方法 ActivityThread#performLaunchActivity 主要干了三件事：1 初始化 Activity 的 Context 和 Application，2 创建 Activity 实例，3 执行 Activity 生命周期方法
```java
Activity#startActivity //启动一个 Activity
-Activity#startActivityForResult
--Instrumentation#execStartActivity //与 ActivityManagerService 进行跨进程通信
---ActivityTaskManager.getService().startActivity //通过 Binder 跨进程通信请求 AMS，然后经过一系列操作后，再由 AMS 通过 Binder 跨进程通信调用触发 ActivityThread 进行处理启动目标 Activity，最终会调用到 ActivityThread#handleLaunchActivity 方法
----ActivityThread#handleLaunchActivity 
```

```java
ActivityThread#handleLaunchActivity //处理 Activity 启动
-ActivityThread#performLaunchActivity //执行完成 Activity 启动，内部进行 Activity 的 Context 的初始化，紧接着进行 Application 的初始化
--Instrumentation#newActivity //内部进行 Activity 的初始化，Instrumentation#newActivity 利用 ClassLoader 类加载器通过反射创建对应的 Activity 实例
--Activity#attach //内部进行 Window（PhoneWindow）的初始化，Window#setWindowManager
--Instrumentation#callActiviyOnCreate
---Activity#performCreate
----Activity#onCreate //最终就调用到了 Activity#onCreate 方法
```

```java
ActivityThread#handleResumeActivity
-ActivityThread#performResumeActivity
--Activity#performResume
---Instrumentation#callActivityOnResume
----Activity#onResume //最终就调用到了 Activity#onResume 方法
```

## Window 初始化流程
- PhoneWindow
```java
ActivityThread#handleLaunchActivity
-ActivityThread#performLaunchActivity
--Instrumentation#newActivity //内部进行 Activity 的初始化，Instrumentation#newActivity 利用 ClassLoader 类加载器通过反射创建对应的 Activity 实例
--Activity#attach //内部进行 Window（PhoneWindow）的初始化，Window#setWindowManager
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
- 负责协调四大组件的生命周期管理、管理 Activity 的任务栈
- 负责应用程序进程的管理和资源调度（AMS 请求 Zygote 进程创建应用程序进程）

## Window、WindowManager 和 WindowManagerService
- Window 包含并管理 View，WindowManager 管理 Window，而 WindowManager 的操作又是通过和 WindowManagerService 交互来实现的

## WindowManager、WindowManagerImpl 和 WindowManagerGlobal
- WindowManager 是一个接口，继承自 ViewManager 接口，WindowManagerImpl 是 WindowManager 的接口实现类
- WindowManager 的作用是对 Window 进行管理，包括增加、更新和删除等操作（定义在 ViewManager 接口里）
- WindowManagerImpl 方法内部又是调用 WindowManagerGlobal 的相关方法（涉及到桥接模式的知识）

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

应用程序进程已启动的情况
- ActivityManagerService -- Binder --> ActivityThread（主线程管理类）
- AMS 通过 Binder 方式请求 ActivityThread 所在进程启动根 Activity
```java
ActivityTaskSupervisor#startSpecificActivity
ActivityTaskSupervisor#realStartActivityLocked //ActivityThread 启动目标 Activity
-ActivityThread$H#sendMessage //通过 Handler 调用 H#sendMessage 方式发送通知
--ActivityThread#handleLaunchActivity //处理根 Activity 启动
---ActivityThread#performLaunchActivity
----Instrumentation#newActivity //利用类加载器完成 Activity 的初始化
----Activity#attach
----Instrumentation#callActivityOnCreate
-----Activity#performCreate
------Activity#onCreate
```

应用程序进程未启动的情况
- ActivityManagerService -- Socket --> Zygote 
- AMS 通过 Socket 方式通知 Zygote 进程去完成应用程序进程的创建
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

## ViewRootImpl
- ViewRootImpl 的创建：ActivityThread#handleResumeActivity -> ActivityThread#performResumeActivity -> WindowManager.addView -> WindowManagerImpl#addView -> WindowManagerGlobal#addView
- ViewRootImpl 是 WindowManager 和 DecorView 之间的桥梁，用来管理 View 的各种事件，包括 invalidate、requestLayout 和 dispatchInputEvent 等
- ViewRootImpl 是 DecorView 的 ViewParent，也正因为这样 View#requestLayout 层层调用最终能调到 ViewRootImpl#requestLayout

## View 绘制机制
- 由 ViewRootImpl 来统一协调管理整个视图树的绘制流程，处理来自系统的输入事件分发和生命周期变化
- 每个 Activity 的 Window 都对应一个 ViewRootImpl 实例（将 ViewRootImpl 和 DecorView 建立关联）

## View 绘制流程
- 关键方法 ViewRootImpl#scheduleTraversals 会触发 ViewRootImpl#performTraversals 去执行 Measure 测量、Layout 布局和 Draw 绘制三大方法
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

## 自定义 View 的基本流程
- 1 重写构造方法，通常是 4 个构成方法，至少重写一个，初始化和获取自定义属性（TypedArray）
- 2 重写 onMeasure，测量 View 大小
- 3 重写 onSizeChanged，确定 View 大小
- 4 重写 onLayout，确定子 View 的布局（ViewGroup）
- 5 重写 onDraw，绘制内容

## 事件分发机制
- MotionEvent 事件产生后，按照 Activity ->  Window -> DectorView -> View 顺序传递的过程就叫事件分发，遵循了一种类似于责任链模式的设计
- 事件序列：通常情况下 MotionEvent.ACTION_DOWN 标志着一个事件序列的开始，而 MotionEvent.ACTION_UP 则标志着一个事件序列的结束，只有当一个视图处理了 ACTION_DOWN 事件，它才会收到后续的 ACTION_MOVE 和 ACTION_UP 事件（一旦对 ACTION_DOWN 事件返回 true，那么后续的 ACTION_MOVE 和 ACTION_UP 事件都会直接传递给它）
- View#dispatchTouchEvent 分发事件
- ViewGroup#onInterceptTouchEvent 拦截事件（只有 ViewGroup 有）
- View#onTouchEvent 处理事件（事件消费）
- OnTouchListener -> OnTouchEvent -> OnClickListener

## 处理 View 的事件冲突
- 外部拦截法：是在父 View 的 onInterceptTouchEvent 方法中进行事件拦截判断
- 内部拦截法：是在子 View 的 dispatchTouchEvent 或 onTouchEvent 方法中，通过调用父视图的 ViewGroup#requestDisallowInterceptTouchEvent 方法来控制父 View 是否拦截事件（父View 是否会跳过 onInterceptTouchEvent）

## View 生命周期
- 在 Activity 的 onResume 方法里是无法获取 View 正确的宽高的
- 官方推荐采用 onWindowFocusChanged(true) 回调来确定当前 View 所在的 Activity 是对用户可见的并且可交互的，所以这里可以获取到 View 正确的宽高
- onAttachedToWindow 进行资源准备初始化，onDetachedFromWindow 进行资源释放取消

## Binder IPC 进程间通信机制
- Binder 是 Android 提供的一种 IPC 进程间通信机制，Binder 是一种基于消息传递的 IPC 机制，基于 C/S 架构
- AIDL 是用来抽象化 Binder IPC 的工具（接口的定义）
- Messenger 是一种轻量级的 IPC 方案，底层是对 AIDL 进行了封装，基于 Handler 和 Message
- 不需要两次拷贝，利用内存映射，只进行了一次拷贝
- 通过 Binder 驱动实现进程间的数据拷贝和通信：当一个进程想要与另一个进程通信时，它会通过 Binder 驱动获取目标进程的 Binder 对象引用（每个 Binder 对象都有一个唯一的 Binder ID），然后通过这个引用发送请求，Binder 驱动会将请求传递给目标进程，目标进程在处理请求后再通过 Binder 驱动返回结果

## Intent
- 当使用 Intent 启动跨进程组件（比如通过 startActivity、startService 或 sendBroadcast 等）时，数据需要通过 Binder 机制传输，传输的数据会被封装为 Parcelable 对象并存储在 Binder 的事务缓冲区中（大小通常为 1MB 左右，且所有 Binder 传输共享该缓冲区，所以实际可用大小约为 500K ~ 800K 以下，建议限制在 100K ~ 200K 以下），所以此时 Intent 传输数据的大小就受 Binder 机制的制约限制

## onNewIntent
可以利用 `setIntent(intent);` 更新 intent

## Context
- 提供资源访问，比如通过 getResources 方法获取资源
- 提供系统服务，比如通过 getSystemService 方法获取系统服务
- 提供应用程序生命周期的管理，比如 Activity、Service 等都是 Context 的子类
- 提供 UI 更新，比如通过 startActivity 方法启动一个新的 Activity

## Fragment
- Fragment 允许将界面分成为好几个区块，从而将模块化和可重用性能力引入 Activity 
- 支持同一功能界面根据屏幕大小不同可以实现两个版本的页面显示样式

## ContentProvider
- ContentProvider 通常会在 Application#onCreate 方法之前初始化，多个 ContentProvider 的初始化顺序遵循 AndroidManifest.xml 文件中的声明顺序

## App Startup
- 很多三方库（比如 LeakCanary）都用 ContentProvider 的小技巧进行 Library 的初始化操作，初始化很多 ContentProvider 一定程度上会影响性能，官方就出了 App Startup 统一到一个 ContentProvider 里去初始化，提供了一个 ContentProvider 来运行所有依赖项的初始化
- 利用 ContentProvider 实现初始化 Library 获取 Context

## Handler 消息机制
- Handler 消息机制是一套设计用来进行线程间通信的机制，主要是用于实现线程切换，能够让子线程间接的去访问 UI 控件，在 Java 层及 Native 层均是由 Handler、Looper、MessageQueue 三者构成
- Handler.postDelayed 修改手机系统时间对延迟消息不会有影响

## Handler
- 消息处理器，负责收发消息，Handler 发 Message 消息到 MessageQueue 消息队列里，而消息队列中的 Message 中存放着的 Handler 引用，在 dispatchMessage 分发消息的时会将消息发到该 Handler 里进行处理
- ActivityThread#main 方法中通过 Looper#prepareMainLooper 调用 Looper#prepare 创建一个 Looper，涉及 ThreadLocal 存取 Looper 对象，而 Looper  的构造方法里创建了一个 MessageQueue
- ActivityThread#main 方法继续执行，里面有个 Looper.loop 方法，内部利用 for 循环在主线程中开启了一个死循环，用它不断得从 MessageQueue 消息队列里取 Message，然后分发消息给 target（Handler）处理

## Looper
- 一个 Handler 对应一个 Looper，每个线程只能有一个 Looper
- loop 方法里的 for 循环  queue.next 取 message  的 target（Handler），然后分发到 callback 或 handleMessage 方法
- queue.next 可能会阻塞（Looper 也就进入了休眠），next 方法里有 nativePollOnce 本地方法，它会调用 Linux 系统的 epoll_wait 实现阻塞等待，直到下一条消息可用为止

## MessageQueue
- 用于存储 Message，内部维护了 Message 的链表（单向链表实现的队列），每次拿取 Message 时，若该 Message 离真正执行还需要一段时间，会通过 nativePollOnce 进入阻塞状态，避免资源的浪费，若存在消息屏障，则会忽略同步消息优先拿取异步消息，从而实现异步消息的优先消费

## 异步消息
- 在创建 Handler 时，Handler 构造函数有一个 async 参数，默认传 false，若 async 传 true 则表示这个 Handler 发出的消息都是异步消息，异步消息会被优先处理

## 同步屏障
- 同步屏障的作用就是为了保证异步消息可以优先执行，ViewRootImpl#scheduleTraversals 方法里就使用了同步屏障，保证 UI 绘制优先执行
- 可以通过 MessageQueue#postSyncBarrier 方法来设置同步屏障，该方法发送了一个没有 target（Handler）的 Message，在 MessageQueue#next 方法中获取消息时，如果发现没有 target 的 Message，则在一定的时间内跳过同步消息，优先执行异步消息，再换句话说，同步屏障为 Handler 消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息

## HandlerThread
- 是一个自带 Looper 的线程，一种具有消息循环的线程，其内部可使用 Handler

## IdleHandler 空闲处理器
- 通常用于在消息队列空闲的时候去执行一些低优先级、轻量级的任务，可以实现一些延迟初始化的应用
- IdleHandler 是一个回调接口，可以通过MessageQueue的addIdleHandler添加实现类。当MessageQueue中的任务暂时处理完了（没有新任务或者下一个任务延时在之后），这个时候会回调这个接口，返回false，那么就会移除它，返回true就会在下次message处理完了的时候继续回调

## SyncBarrier 同步屏障
- 同步屏障的特点是 msg.target = null，就是没有关联 Handler 的消息，相当于一个标志位，通过 MessageQueue#postSyncBarrier 发送，通过 MessageQueue#removeSyncBarrier 进行移除
- ViewRootImpl#scheduleTraversals 方法就是通过  mHandler.getLooper().getQueue().postSyncBarrier()  添加同步屏障以确保来保证 UI 绘制优先执行

## Init、Zygote 和 SystemServer 进程
- Init 由 Linux 系统内核启动，解析 init.rc 文件后 fork 生成 Zygote 进程
- Zygote 由 Init 进程启动，是所有 App 应用程序进程的父进程
- SystemServer 负责启动各种系统核心服务

## IntentService
- IntentService是一个抽象类，继承自Service，内部存在一个 ServiceHandler（Handler）和HandlerThread（Thread），用来来处理耗时操作
- 单线程执行：Service 默认在主线程中运行，而 IntentService 是运行在一个子线程中，不会阻塞主线程，默认无需去手动管理线程
- 自动停止：当执行完任务之后 IntentService 会自动停止
- 多次启动IntentService，每一个耗时操作都会以工作队列的形式在IntentService的onHandleIntent回调中执行，并且每次执行一个工作线程
本质是封装了一个 HandlerThread 和 Handler 的 Service  是一种异步、会自动停止的服务，内部采用HandlerThread

## JobScheduler

## WorkManager

## 后台任务解决方案
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

## ThreadPoolExecutor 线程池工作原理
- 如果线程池中的线程数量未达到 corePoolSize 核心线程的数量，那么会直接创建一个核心线程来执行任务（即使此时线程池中存在空闲线程）
- 如果线程池中的线程数量已经达到 corePoolSize 核心线程的数量，那么任务会被插入 workQueue 任务队列中排队等待调度执行
- 如果 workQueue 任务队列已满，但是线程数量未达到 maxPoolSize 最大线程数，那么会立刻创建一个非核心线程来执行任务
- 如果 workQueue 任务队列已满，且已经达到 maxPoolSize 最大线程数，那么就采用拒绝策略来响应任务（丢弃任务并抛出异常、直接丢弃后来的任务不抛出异常和丢弃阻塞队列中最老的任务等）

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

lifecycle.repeatOnLifecycle 和 Flow 协力能够打造出高效的类似 LiveData 的通信机制
```kotlin
lifecycleScope.launch {
        //当 Fragment 处于 STARTED 状态时会开始收集数据，并且在 RESUMED 状态时保持收集，最终在 Fragment 进入 STOPPED 状态时结束收集过程
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            uiStateFlow.collect { uiState ->
                //updateUi(uiState)
            }
        }
    }
```

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

## 动画
- Frame 逐帧动画是通过按顺序快速播放一系列图像（通常称为帧）来形成动画效果的技术
- Tween 补间动画不会改变视图的实际属性，只是改变了视图在视觉上呈现动画效果，所以不适合用在有事件交互的场景
- Property 属性动画能对对象的任意属性（比如位置、大小、颜色、透明度等）进行动画处理，如果需要对自定义属性实现动画效果需要保证自定义属性对应的 getter 和 setter 方法是可被访问的

## Lottie 动画
Lottie 动画依托于 ValueAnimator 属性动画，动画更新的监听不断回调，执行 LottieDrawable#setProgress 的最终会触发了每个 layer 的 invalidateSelf，这都会让 LottieDrawable 重新绘制，然后重走一遍绘制流程，这样随着 animator 动画的进行，LottieDrawable 不断的绘制，就展现出了一个完整的动画

## 扩大 View 点击区域
- 增加 padding
- TouchDelegate
- getLocationOnScreen + onTouchEvent

## 命令式编程和响应式编程的区别
- 命令式编程强调步骤和过程，侧重于“怎么做”
- 响应式编程强调以声明式的方式处理异步数据流，关注于定义数据流及其变换规则，而不是控制执行流程，更侧重于“做什么”

## 内存泄漏
- 内存泄漏：本该被 GC 回收的内存实际没有被回收
- 根本原因：短生命周期的对象被长生命周期的对象持有引用，导致短生命周期对象的内存在其生命周期结束后无法被 GC 回收，从而导致了内存泄露
- 非静态内部类或匿名内部类都会隐式地持有外部类的引用，都可以访问外部类的变量
- 1 减少单例、静态对象、非静态内部类或匿名内部类的使用
- 2 使用 WeakReference 弱引用来避免强引用导致的内存泄漏
- 3 注意 Handler 导致的内存泄漏，及时在 onStop 或 onDestroy 时候用 removeCallbacksAndMessages 移除消息
- 4 针对线程相关及时做好取消、终止等工作（比如子线程耗时操作运行仍未结束）
- 5 针对资源性对象（比如数据库、File 文件和 Cursor 等）、WebView 和动画等需要及时做好回收关闭工作，做好 BroadcastReceiver 广播、监听器和观察者等的移除反注册工作，Service 服务执行完成后记得关闭

## 内存抖动
- 内存抖动：主要是内存波动图类似锯齿状，代表存在频繁地内存开辟和回收（在短时间内有大量的对象被创建和销毁，比如在循环或高频回调中大量创建对象），触发频繁的 GC，容易导致内存碎片，也容易引发卡顿
- 1 避免在循环或高频回调中创建大量对象（比如在 onDraw、onTouchEvent 中创建对象）
- 2 使用对象缓存池（比如 Message#obtain、Recyclerview#RecycledViewPool），减少对象的频繁创建和销毁
- 3 使用 StringBuilder、StringBuffer 代替 String 字符串拼接 
- 4 使用合理的数据结构，同时在能够预知对象大概数量的情况下，显式去设置初始化容量，减少触发数据结构内部的扩容操作（比如 HashMap 和 ArrayList 预分配的集合容量）

## 内存溢出
- Out Of Memory 内存溢出（内存不足）：指程序在申请内存时，当下应用内存没有足够的内存空间供分配使用，则会导致程序崩溃或异常终止的现象，长期的内存泄露积压会导致 OOM，同时如果一次性开辟太大的数组或者加载过大的图片也很容易导致 OOM
- 1 内存中存在大量的强引用对象（集合类未清理、静态变量滥用），推荐选择合适的引用方式创建对象（对于不经常访问的对象，可以使用 WeakReference 弱引用或 SoftReference 软引用代替强引用，以便在内存不足时能够被回收）
- 2 优化数据结构：使用 ArrayMap 代替 HashMap，使用 SparseArray 代替 `HashMap<Integer, Object>`，避免自动装箱拆箱，减少 Entry 对象开销
- 3 使用懒加载：延迟加载不必要的资源，可以提高应用的启动速度和内存使用效率
- 4 避免使用 Enum
- 5 使用线程池处理异步任务
- 6 使用 DiffUtil 进行增量更新，避免全局刷新
- 7 Bitmap 内存优化
  - 图片压缩：在加载图片前进行压缩，尽量避免加载原图，减少内存占用，如果加载长图，按需解码部分区域，避免一次性加载全图
  - 图片缓存：使用 LruCache 等缓存机制，尽量避免重复加载图片
  - 选择合适的图片格式（比如 WebP 格式）
  - 对于复杂的图像处理任务，使用 RenderScript 来加速计算

## LeakCanary
- LeakCanary 原理是使用 WeakReference 弱引用来跟踪 Activity，并结合 GC 机制判断 Activity 是否被回收，从而识别潜在的内存泄漏问题

## BlockCanary
- 也是利用 MainLooper#setMessageLogging 接管系统的 logging Printer 计算时间差来检测卡顿

## Choreographer 编舞者
- 进行掉帧检测，在每一帧回调时计算帧间隔时间，若间隔时间超过 16 毫秒，就认为可能存在掉帧的情况

## FrameMetrics
- Android 高版本直接获取帧率相关数据信息

## 腾讯 Matrix
- 结合了 Choreographer 和 Looper 监控方案，分析 UI 线程的 Message 执行耗时

## JankStats 卡顿统计（Metrics 指标库）
- Android 高版本采用 FrameMetrics，Android 低版本采用 Choreographer

## MediaCodec
- MediaCodec 类可以访问底层的编解码器，用于编解码音视频。它提供了一种直接处理压缩数据（例如 AAC、MP3 和 H.264 等）的方法。
- MediaCodec 编码摄像头画面 MediaMuxer 合并生成 MP4 文件
- MediaCodec 解码播放 MediaExtractor 提取 MP4 文件的内容
- AudioTrack 播放音轨
- 使用 MediaCodec，你可以进行诸如视频播放、视频编辑、实时通信等操作。要使用 MediaCodec 进行解码，通常需要先配置媒体格式，然后调用 configure() 方法，接着是 start()，之后可以通过 dequeueInputBuffer() 和 queueInputBuffer() 来输入数据，并通过 dequeueOutputBuffer() 获取输出的数据

## 编码格式和容器格式
- 编码格式： H.264/AVC H.265/HEVC MP3 AAC
- 容器格式： MP4 MKV 

## HTTP 超文本传输协议
- HTTP 是一种以明文形式传输数据，简单、灵活可扩展的，基于 TCP/IP 协议传输数据，是可靠的传输协议
- HTTPS 超文本传输安全协议，通过增加 SSL/TLS 协议来加密 HTTP 数据传输，利用加密、完整性校验和身份验证机制确保数据在传输中不被窃取或篡改
- HTTP/2 作为 HTTP 协议的第 2 个主要版本，引入了二进制分帧（二进制格式）、多路复用、头部压缩、服务器推送、请求优先级等特性

## 浏览器中输入一个 URL  然后回车访问，这个过程中间发生了什么？
- DNS 解析，把网址解析找到域名对应的 IP 地址
- 客户端发起 HTTP 请求，会进行 TCP 三次握手建立连接，服务端发送报文给客户端
- 客户端收到报文解析得到 HTML页面等数据，然后构建 DOM 树，再构造呈现树并绘制到浏览器页面上面

## TCP 三次握手
- 客户端发送一个 SYN（同步序列编号）报文到服务器以发起连接
- 服务器收到 SYN 报文后，回复一个 SYN-ACK（同步和确认）报文
- 客户端收到 SYN-ACK 报文后，发送一个 ACK（确认）报文作为响应，完成握手

## TCP 四次挥手
- 客户端发送一个 FIN（结束）报文到服务器请求关闭连接
- 服务器收到 FIN 报文后，发送一个 ACK 报文作为响应
- 服务器发送一个 FIN 报文到客户端请求关闭连接
- 客户端收到 FIN 报文后，发送一个 ACK 报文作为响应，完成挥手

## TCP/IP 协议
- 四层网络模型：应用层、传输层、网络层（互联网层）和网络接口层
- 五层网络模型：应用层、传输层、网络层（互联网层）、数据链路层和物理层

## UDP 协议
UDP 协议，和 TCP 的区别
前后台传输数据需要用密钥对数据加密，那加密过程应该放在哪个位置

## OkHttp
OkHttp 中可以通过 Interceptor 拦截器实现重试机制，使用 GZIP 压缩请求和响应数据，减少传输数据量

## 缓存机制
三级缓存：内存、硬盘、网络
LRU：Least Recently Used 最近最少使用，常用于页面置换算法、常用作缓存淘汰策略

Android 缓存机制
> 狭义上是指针对网络的缓存
> 广义上指的是对数据的复用
 LRU如何实现  如何自己实现一个LRUCache？Android里面的LRUCache是如何实现的

## 缓存策略
- FIFO
- LFU Least Frequently Used 最不经常使用
- LRU 最近最少使用

## LruCache

## RecyclerView 缓存机制
- ViewHolder 主要解决了 ItemView 复用的问题，减少重复 findViewById 的消耗
四级缓存：
    - 屏幕内缓存：mAttachedScrap（数据未改变）和 mChangedScrap（数据已改变）临时保存屏幕内的 ViewHolder
    - 屏幕外缓存：mCachedViews 最近被移出屏幕的缓存，默认缓存数量是 2 个，本质上是一个 ArrayList，当 mCachedViews 满了之后，新移除的 ViewHolder 会被放入 RecycledViewPool 中
    - 自定义缓存：ViewCacheExtension
    - 共享缓存池：RecycledViewPool，默认缓存数量是 5 个，本质上是一个 SparseArray，其缓存的 ViewHolder 是全新的，复用时需要重新绑定数据（重新调用 bindViewHolder）

## DiffUtil
- 使用 DiffUtil 进行增量更新，避免全局刷新

## Alibaba ARouter
- 用于实现 Android 组件化开发的路由框架，它通过实现不同模块之间的页面跳转和参数传递等功能，使得不同的业务模块之间进行解耦，可以更加独立地开发和维护各个模块
- 可以使 Android 应用的架构更加清晰、可维护性更高
- 支持降级处理，如果目标页面或服务不可用，可以进行降级处理，提高应用的稳定性
- 拥有控制拦截能力，可以配置拦截器来控制页面跳转
- ARouter 路由跳转实际上还是调用了 startActivity 的跳转（基于注解处理、动态生成代码、使用反射加载）

## MVVM 和 MVI
- MVVM 侧重双向数据绑定（View <-> ViewModel），双向绑定一般由 DataBinding 实现，但是大部分开发者都会使用 LiveData、Flow 来观察数据变化，通常为了保证数据流的单向流动性（非必须），会向外暴露不可变的 LiveData，所以 ViewModel 里一个 State 状态会存在两个 LiveData，一个可变的一个不可变的，如果状态很多，LiveData 数量就会大大地增多
- MVI 是 MVVM 的升级方案，侧重单向数据流（View -> UIIntent -> ViewModel -> UIState -> View）约束，强调不可变状态（State 实例是不可变的，每次状态更新时都会创建新的 State 对象）和单一（唯一可信）数据源，MVI 里的 Model 侧重指 UIState 状态，UIState 用于封装页面状态和数据（比如页面加载状态、控件位置等），对 State 进行了集中状态管理（这样一来 LiveData 就只需要一份了，一个可变的一个不可变的），UIIntent 用于包装用户的操作意向（意图）发送给 ViewModel 进行数据请求，另外可以额外引入 UIEvent 来处理一次性的事件
- UIState、UIIntent 通常都可以采用密闭类（密封类）
- Activity/Fragment 里观察 UiState 状态，然后通过 when 判断处理对应逻辑
- ViewModel 里可以提供集中统一处理 UIIntent 的入口方法
- MVI 中所有 UIState 状态都可以通过一个 LiveData 来管理，也随之而来引出新的问题，就是页面不支持局部刷新
- 由于 UIState 状态是不变的，因此每当 UIState 需要更新时都要创建新对象替代老对象，这会带来一定内存开销
```kotlin
//ViewModel 里的模板代码
private val _loading: MutableLiveData<String> = MutableLiveData()
val loading: LiveData<String> = _loading
```

## 架构
- 使用 ViewModel 而非 AndroidViewModel：不建议使用 AndroidViewModel，不应该在 ViewModel 中使用 Application 类，应该将依赖项移至界面层或数据层
- 如果是要共享数据的话，应该在业务层或者repo层就做了，不应该在vm这层来做。
- UseCase 应该是 Main-safe 的，即可以在主线程安全的调用，其中的耗时处理应该自动切换到后台线程

## 命名规范
- Repository 接口
NewsRepository（以往是 INewsRepository、NewsRepositoryInterface）

- Repository 实现 
DefaultNewsRepository（默认）、OfflineFirstNewsRepository、FakeNewsRepository（以往是 NewsRepository、NewsRepositoryImpl）

data
- local
-- entitity
-- dao
-- XxxDatabase
- network/remote
-- NetworkApi
-- models
- repository
- model

## Acoustic Echo Cancellation 回声消除
- 麦克风采集到的音频通过扬声器（喇叭）播放出来后又被采集进去，从而产生了回声或啸叫声
- 理想上：假设 A，B 两端，A 端把听到的声音（混合信号）中删去 A 之前的回声（参考传出的信号），就只剩下了 B 的语音
- 实际上：但 A 端是无法直接参考使用之前传出的信号的（差异较大），而 A 之前的回声又不可能凭空臆想出来
- 所以采用数学算法模拟出这个 A 之前的回声，从混合信号中减去这个模拟回声信号，从而达到回声消除的效果
- 硬件回声消除算法和开源软件回声消除算法（Opus、WebRTC-AEC 模块）

## Hook 技术
- 利用 Java 反射实现，动态代理，ClassLoader 
- 静态变量或者单例对象，尽量 Hook public 的对象和方法
- 用代理对象替换原始对象，接口可以用动态代理
- Hook 的时机还是尽量要早，API 版本比较多，方法和类可能不一样，要做好 API 的兼容工作

## AOP 面向切面编程
- Eclipse AspectJ

## Glide
- 生命周期绑定（采用没有 UI 的 Fragment 来管理）
- 缓存设计（弱引用的缓存、LruCache 的内存缓存、DiskLruCache 磁盘缓存），支持缓存不同大小尺寸
- Glide 内部的 KeyPool 是基于 Queue 队列实现的，Glide 内部的 BitmapPool 是基于带了 LRU 算法的 Map 实现的


