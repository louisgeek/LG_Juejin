# Android 知识点

GC不是在Activity销毁时就立即进行的。GC是每隔一段时间就自发进行一次，而且这次回收不了、下次还有机会。这就是说，即便你的新线程里存在耗时的代码（之所以强调新线程，是因为阻塞主线程会直接报错），即便你那耗时的代码在其外部的Activity销毁之后仍然运行，只要这代码别整个几分钟都停不下来，那就算稍微晚几秒被回收也无所谓。为什么要强调这个呢？因为这样你就不用非得把一些有可能在Activity销毁后继续运行但耗时较短的内部类设为静态了
 
Lottie 动画依托于 ValueAnimator 属性动画，动画更新的监听不断回调，执行 LottieDrawable#setProgress 的最终会触发了每个 layer 的 invalidateSelf，这都会让 LottieDrawable 重新绘制，然后重走一遍绘制流程，这样随着 animator 动画的进行，lottieDrawable 不断的绘制，就展现出了一个完整的动画


 
协程 catch 原理

为什么 try catch 在协程外面不行

juc 集合

vysnc 事件。重复绘制如何触发
 
抓包 ban 网站
profiler 内存分类
滑动冲突
饼图

事件分发 cancel 
invitations 

lock 原理

emit  try c

在 Flow 中，背压的管理是隐式的，由协程的挂起（suspend）机制来实现，当 Flow 的生产者发射数据时，如果消费者还没有准备好接收下一条数据，生产者会挂起，直到消费者准备好

supcaleable  回调结束在哪


Flow Channel 



## 你的职业规划？
- 评估自己感兴趣的方向和技能
- 规划阶段性职业目标
- 技能提升：通过学习、实践和反馈
- 总结目标达成情况并适当调整



一般遇到困难的解决方案是什么




app质量如何控制？



技术选型是如何做的

讲一下你的技术栈


你最擅长的点


你做过的最烂的一件事是什么？最好、最自豪的一件呢？最好或者最自豪的项目




RecycleView每一级缓存的作用以及使用场景

mChangedScrap:存放可见范围内item的更新，并且调用了notifyItemChanged、notifyItemRangeChanged这类方法通知更新
mAttachedScrap:临时存放已添加或者已删除item,使用场景是列表滚动，或者删除item后调用notifyItemRemoved通知更新
mCachedViews：滚动过程中，存放未被重新使用且状态无变化的item,一般用于列表滚动过程中将已经移出屏幕的item快速且无加载得滑动回来
mViewCacheExtension：主要提供给开发人员自定义缓存层级
mRecyclerPool：按照不同itemType分别存放超出mCachedViews限制并移出屏幕的item

 


SSL/TLS 协议使用公钥和私钥对数据进行加密和解密，确保数据在传输过程中不被窃取或篡改。
它使用公钥加密来确保通信双方的身份验证，然后使用对称加密来保护数据的隐私性和完整性
SSL/TLS 协议使用消息认证码（MAC）来保证数据的完整性



短连接
长连接




HTTP/2 引入了二进制格式来打包和传输客户端与服务器之间的数据，这有助于降低解析难度并提高协议的健壮性
    这使得数据传输更快，解析更高效，减少了传输过程中的开销和延迟
 这提高了效率和安全性
通过这种方式，服务器和客户端可以更高效地解析和处理数据，实现多路复用。



px（pixel），可以理解为一个最小图像单元（只能涂一个颜色）的小方块，就是1px，是一小块面积，但是一般并不强调面积的大小，只是说这是一个最小单元。


由于 HTTP 无状态的特点，每次 HTTP 请求都需要通过 TCP 进行三次握手保证双方具有接收和发送的能力，即在信道不可靠情况下, 要保证可靠的数据传输，就需要至少三次通信，即双方确认自己与对方的发送与接收是否正常





#### Handler 发送延迟消息没有及时发完
- Activity - 匿名 Handler - Message - MessageQueue - Looper 
- 推断：主线程 Looper 一直在 loop ，Looper 持有 MessageQueue，MessageQueue 持有 Message，Message 持有 Handler，匿名 Handler 持有 Activity 引用
 



### Heap Dump
- Android Studio 自带的 Heap Dump 堆转储工具
Profiler - Sessions 里选择指定进程 - Memory 
有两个按钮，force garbage collection GC 按钮和 Dump Java heap 捕获堆转储文件按钮
前者能触发一次手动 GC ，一般我们在我们捕获堆转储文件之前，推荐点一下 GC，能把一些弱引用给回收，防止给我们的分析带来干扰，后者可以生成 hprof 文件，这个文件会展示 Java 堆的使用情况，点击这个按钮后，Android Studio 会帮我们生成这个堆转储文件并且可以进行分析

分析内存快照
 查找可疑对象
 特定类型的对象或者引用路径




 4.优化布局层次，重用，需要时加载 ViewStub，减少内存消耗；



TTS 文本转成语言 Text To Speech

libusb 可以通过 jni 获取 bus dev Manafacturer Product SerialNumber (libusb_context)



## Activity 启动流程
startActivity最终都会调用startActivityForResult，
通过ActivityManagerProxy调用system_server进程中
ActivityManagerService的startActvity方法，
如果需要启动的Activity所在进程未启动，
则调用Zygote孵化应用进程，
进程创建后会调用应用的ActivityThread的main方法，
main方法调用attach方法将应用进程绑定到

ActivityManagerService（保存应用的ApplicationThread的代理对象）
并开启loop循环接收消息。ActivityManagerService
通过ApplicationThread的代理发送Message通知启动Activity，
ActivityThread内部Handler处理handleLaunchActivity，
依次调用performLaunchActivity，
handleResumeActivity（即activity的onCreate，onStart，onResume）


## Handler
```java
static volatile Handler sMainThreadHandler;
//ActivityThread#main
public static void main(String[] args) {
	Looper.prepareMainLooper();

	ActivityThread thread = new ActivityThread();
	thread.attach(false, startSeq);

	if (sMainThreadHandler == null) {
 	  sMainThreadHandler = thread.getHandler();
	}


	Looper.loop();
    //主线程的死循环一旦退出，就会抛异常，本意就是在正常情况下得让该方法一直处于死循环状态
	throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

```java
 public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                //只能创建一次，主线程有且仅有一个 Looper
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```
，而由于哈希值数组是有序的，且键值对数组按照哈希值数组的顺序存储，因此插入顺序得以保留  保持键值对的插入顺序！！！
 Asset 资产
使用JobScheduler和WorkManager：合理安排后台任务，减少唤醒次数
通过 Android Profiler 分析内存与CPU占用
TraceView用于分析应用的执行流程，找出耗时操作
Systrace用于系统级的性能分析，帮助开发者优化系统启动过程
使用AlarmManager的弹性机制：使用setInexactRepeating来减少唤醒的精确度

OkHttp中，可以通过 Interceptor 拦截器实现重试机制，使用GZIP压缩请求和响应数据，减少传输数据量

### ThreadLocal 是什么?

- ThreadLocal 是一个线程内部的数据存储类，在指定的线程中存取数据，别的线程无法获取该线程的数据，实现线程的数据隔离

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有指定线程中可以获取到存储的数据，对于其它线程来说无法获取到数据。一般开发情景中我们用不到ThreadLocal，但是在一些非常复杂或者特殊的情景中， 通过ThreadLocal可以轻松地实现一些看起来很复杂的功能。

在Android的系统机制中，以下机制中都使用到了ThreadLocal：Looper、ActivityThread、ActivityManagerService。

**一般来说，当我们管理数据的时候是以线程为作用域并且不同线程管理不同的数据副本的时候，就可以考虑使用ThreadLocal。ThreadLocal的作用就是提供了一个全局的哈希表，用于实现指定线程的数据控制。**

具体到ThreadLocal的使用场景，一般可以概况为，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。



### 主线程的消息队列是否有上限?





### 为什么不允许子线程直接去操作 UI 控件？不采取对 UI 加锁的方式解决？

因为 UI 控件不是线程安全的，子线程去操作可能会导致 UI 的不稳定和不确定性

加锁会让 UI 访问的逻辑变得复杂，而且每次加锁会降低 UI 访问的效率


 
 









### 可以让自己发送的消息优先被执行吗？原理是什么？

这个问题，我感觉只能说：在有同步屏障的情况下是可以的。

同步屏障作用：在含有同步屏障的消息队列，会及时的屏蔽消息队列中所有同步消息的分发，放行异步消息的分发。

在含有同步屏障的情况，我可以将自己的消息设置为异步消息，可以起到优先被执行的效果。



### Looper 主线程存在死循环，为什么没有卡死？没有发生ANR？

先说下ANR：5秒内无法响应屏幕触摸事件或键盘输入事件；广播的onReceive()函数时10秒没有处理完成；前台服务20秒内，后台服务在200秒内没有执行完毕；ContentProvider的publish在10s内没进行完。所以大致上Loop死循环和ANR联系不大，问了个正确的废话，所以触发事件后，耗时操作还是要放在子线程处理，handler将数据通讯到主线程，进行相关处理。

线程实质上是一段可运行的代码片，运行完之后，线程就会自动销毁。当然，我们肯定不希望主线程被over，所以整一个死循环让线程保活。

为什么没被卡死：在事件分发里面分析了，在获取消息的next()方法中，如果没有消息，会触发nativePollOnce方法进入线程休眠状态，释放CPU资源，MessageQueue中有个原生方法nativeWake方法，可以解除nativePollOnce的休眠状态，ok，咱们在这俩个方法的基础上来给出答案

- 当消息队列中消息为空时，触发MessageQueue中的nativePollOnce方法，线程休眠，释放CPU资源
- 消息插入消息队列，会触发nativeWake唤醒方法，解除主线程的休眠状态

- 当插入消息到消息队列中，为消息队列头结点的时候，会触发唤醒方法
- 当插入消息到消息队列中，在头结点之后，链中位置的时候，不会触发唤醒方法

- 综上：消息队列为空，会阻塞主线程，释放资源；消息队列为空，插入消息时候，会触发唤醒机制

- 这套逻辑能保证主线程最大程度利用CPU资源，且能及时休眠自身，不会造成资源浪费

- 本质上，主线程的运行，整体上都是以事件（Message）为驱动的





## Binder IPC 进程间通信
### Linux 进程间通信方式：管道 消息队列 共享内存 套接字 信号量

### !!!!Binder IPC 通信大概分了几层
一、Java 层：供应用程序开发者使用的 Java 接口层，通过这一层可以方便地进行进程间通信。
二、Native 层 JavaBBinder 和 BpBinder：Native 层中，，用于实现 Java 层与内核层之间的通信转换。
三、Native 层 Binder 驱动：。
四、服务端和客户端通信层：服务端实现具体的服务逻辑并通过 Binder 机制提供给客户端调用，客户端通过 Binder 代理与服务端进行通信。

Framework 框架层：这一层主要包含Java Binder和JNI（Java Native Interface）部分，用于上层应用程序与Binder框架的交互。
Native 层：创建 Service Manager 以及 BBinder、BpBinder 模型，负责实际的 IPC 通信过程，Java 层的 Binder 对象对应 Native 层的 JavaBBinder 对象，Java 层的 BinderProxy 对象对应 Native 层的 BpBinder 对象
Kernal 层：对应就是 Binder 驱动，位于 Linux 内核中，负责处理进程间通信的底层实现，管理 Binder 实体对象和引用，以及处理进程间的数据传输和同步等操作


### 哪些场景会涉及到 Binder IPC 通信
- 1 当应用程序需要访问系统服务（如 ActivityManager、PackageManager、WindowManager 和 ContentProvider等）时，会通过 Binder 机制与系统服务进行通信
    - ActivityManager.getService 获取 ActivityManagerService 实例
    - Context.getContentResolver 获取 ContentResolver 实例
- 2 startActivity、startService、bindService 和 registerReceiver 等方法会涉及与系统的 ActivityManagerService 等服务进行 Binder 通信
- 3 通过 ContentResolver.query 等方法查询数据时，会与对应的 ContentProvider 进行 Binder 通信以获取数据
- 4 生命周期回调，比如 onCreate , onStart , onResume 等
- 5 插件化框架中的通信应用
- 6 AIDL 就是通过 binder 实现跨进程通信





## 组件化


## 插件化



Gradle 的实现，gradle中task的生命周期
WebSocket与socket的区别？
vm的运行时数据结构。栈帧中会有什么异常？方法区里面存放的是什么数据



 

### View 绘制入口


创建 ViewRootImpl 建立 WindowManager 和 DecorView 之间的连接
ViewRootImpl 的 performTraversals
performMeasure、performLayout、performDraw
依次执行 measure、layout、draw 三大流程
measure用来对View进行测量
layout 负责将子元素在父元素中的位置即真实宽高以及四个顶点位置
draw 负责将View绘制出来
 
View的绘制从ActivityThread类中Handler的处理RESUME_ACTIVITY事件开始，在执行performResumeActivity之后，

创建Window以及DecorView并调用WindowManager的addView方法添加到屏幕上，
addView又调用ViewRootImpl的setView方法，
最终执行performTraversals方法，依次执行performMeasure，performLayout，performDraw。也就是view绘制的三大过程。

 measure过程测量view的视图大小，最终需要调用setMeasuredDimension方法设置测量的结果，如果是ViewGroup需要调用measureChildren或者measureChild方法进而计算自己的大小。

 layout过程是摆放view的过程，View不需要实现，通常由ViewGroup实现，在实现onLayout时可以通过getMeasuredWidth等方法获取measure过程测量的结果进行摆放。

 draw过程先是绘制背景，其次调用onDraw()方法绘制view的内容，再然后调用dispatchDraw()调用子view的draw方法，最后绘制滚动条。ViewGroup默认不会执行onDraw方法，如果复写了onDraw(Canvas)方法，需要调用 setWillNotDraw(false);清楚不需要绘制的标记。



 







### requestLayout 流程


- View#measure
ViewRootImpl#performLayout
- View#layout
- 执行调用过 requestLayout 的 View 的 measure 和 layout
- 将还没有执行的 requestLayout 加到队列中，下一次 frame 中进行执行
ViewRootImpl#performDraw
- ViewRootImpl#draw
#### Layout

View 用 View#layout 确定自身位置

View   View#layout

ViewGroup 需要调 onLayout 确定子 View 的位置

ViewGroup   View#layout ->  View#onLayout 

### Draw

画背景

onDraw 进行绘制自己

dispatchDraw 绘制子 View





### Measure

#### MeasureSpec

int 32 位  2 位 SpecMode + 30 位 SpecSize 

SpecMode 

- UnSpecified 
- Exactly
- at_most



DecorView 的 MeasureSpec 由 Window 的大小和自己的 LayoutParams 决定

View 的 MeasureSpec 由父容器的大小和自己的 LayoutParams 决定

 View -> onMeasure -> setdis -> setdisraw

ViewGroup -> onMeasure(子 View ) -> setdis -> setdisraw








### 获取 LayoutInflater 实例的方式
- LayoutInflater 布局解析器，用于解析布局文件后动态生成 View
- 是一个抽象类
- 是一个系统服务
1 Context#getSystemService
2 LayoutInflater#from
3 Activity#getLayoutInflater
三种获取实例的方法最终都是通过 Context#getSystemService 方法实现的，里面涉及到单例模式
Activity#setContentView 方法内部也用到了 LayoutInflater 进行布局解析
LayoutInflater 内部采用了 org.xmlpull 解析器实现 xml 解析



### LayoutInflater#inflate 的流程
- 有一个 View#inflate 方法，就是调用 LayoutInflater#inflate 两个参数的方法
  内部会自动 addView 所以不适合用在 Adapter 和加载 Fragment 布局等情况
1 LayoutInflater 有四个重载方法，其中两个针对 xml 布局，另外两个是针对现成的 XmlResourceParser
2 如果是 xml 布局文件，可以通过 Resources#getLayout 把 xml 布局文件解析加载得到 XmlResourceParser
3 因为 XmlResourceParser 包含属性信息，所以利用 Xml#asAttributeSet 将其转化成 AttributeSet 供后续使用
4 通过 advanceToRootNode 方法找到根标签，如遇到的不是 <merge/> 标签就通过 createViewFromTag 方法把根标签对应类通过反射的方式创建出来，然后调用 rInflateChildren 方法，而它就是调用了 rInflate 方法，从而实现了递归，而 rInflate 内部也调用了 createViewFromTag ，所以用递归调用的方式把子 View 一个个创建出来并通过 addView 加入其父 View 中，如遇到 <merge/> 标签就直接调用 rInflate 先进行处理一次，最终形成一个视图树
5 整个流程中还涉及到反射、ClassLoader、换肤等知识点可以展开



 


 






### WMS 作用
- WindowManager 具体的操作都通过 WMS 处理实现的，所以 WMS 主要功能是 View 的最终管理者
- 和 WindowManager 之间是通过 Binder 实现跨进程通信的
- WMS 负责窗口管理，关联 WindowManager
- WMS 负责窗口动画管理，关联 WindowAnimator
- 输入系统的中转站，关联 InputManagerService
- Surface 管理，关联 SurfaceFlinger


 
### Window 的类型
Application Window 应用程序窗口

- 如 Activity

Sub Window 子窗口

- 不能独立存在，需要依附在其他窗口存在，如 PopupWindow

System Window 系统窗口

- 如 Toast、系统音量条窗口



 
 

 

### Context#getSystemService

 其实就是 ContextImpl#getSystemService 内部通过从 ArrayMap 中获取

ArrayMap 里的值是在静态代码块中通过 SystemServiceRegistry#registerService 存入的

```java
static {
...
    registerService(Context.ACTIVITY_SERVICE, ActivityManager.class,
                new CachedServiceFetcher<ActivityManager>() {
            @Override
            public ActivityManager createService(ContextImpl ctx) {
                return new ActivityManager(ctx.getOuterContext(), ctx.mMainThread.getHandler());
            }});
...
registerService(Context.WINDOW_SERVICE, WindowManager.class,
                new CachedServiceFetcher<WindowManager>() {
            @Override
            public WindowManager createService(ContextImpl ctx) {
                //这里是 Impl 实现类，一个参数
                return new WindowManagerImpl(ctx);
            }});
...
}
```


 



 

### Android 系统启动流程
```
接通电源，按下启动按钮，ROM 中的引导芯片代码开始执行，会加载 BootLoader 到 RAM 中执行
BootLoader 引导程序负责拉起操作系统，引导完后 Linux 内核启动进行加载驱动和启动系统服务等系统设置，查找 init.rc 文件，启动 Init 进程
初始化和启动属性服务，启动 Zygote 进程
创建 Java 虚拟机并注册 JNI 方法，创建名为 "zygote" 的 Server Socket，启动 SystemServer 进程
创建 Binder 线程池和 SystemServiceManager，启动各种系统服务
启动 Launcher ，显示已安装 APP 列表
```

 
#### init 进程启动流程

init.rc 配置文件中， AIL 语句中 Import 语句引入  init.${ro.zygote}.rc 文件，如 init.zygote64.rc

```
init.cpp#main
- mount 挂载启动所需的文件目录
- property_init
- signal_handler_init 子进程信号处理函数，会收到进程终止的消息，防止 init 进程的子进程成为僵尸进程
- start_property_service
- parser 解析 init.rc 配置文件
-- AIL 语句中 Service 语句完成 service.cpp#ServiceManager#AddService 加入配置的服务到 vector  列表
-- AIL 语句中 Action 语句完成 builtins.cpp#ServiceManager 进行 ForEach 遍历得到 Service,然后执行 service.cpp#Start
--- 利用 service.cpp#folk 函数创建 Service 子进程，service.cpp#execve 函数启动该子进程,并会进入到 Service 的 main 函数里，ps：如果 service 是 Zygote 类型，就相当于进入了 Zygote 进程的 main 函数里
```

init 进程

- Android 系统用户空间的第一个进程

- 创建和挂载启动所需的文件目录

- 初始化和启动属性服务

- 解析 init.rc 配置文件，并启动 Zygote 进程（ init.zygote64.rc 系列配置文件）



##### 属性服务

类似 Windows 注册表管理器，以键值对的形式存储系统和软件的信息

```
init.cpp#main 中的 start_property_service
- property_service.cpp#create_socket 创建非阻塞的 Socket 返回 property_set_fd 
- property_service.cpp#listen 函数传入 property_set_fd，形成 server 型 Socket
- property_service.cpp#register_epllo_handler 利用 linux 的 epoll 来监听 property_set_fd ，并通过 property_service.cpp#handle_property_set_fd 函数来处理，内部又调用了 property_service.cpp#handle_property_set 函数进行处理
-- 通过是否有 "ctl."前缀来分成两种类型，一种控制 handle_control_message ，一种普通 property_set 
```



##### Zygote 进程启动流程

```
接上文【init 进程启动流程】最后已经进入到 Service 的 main 函数里了
如果 init 进程解析的是 init.zygote64.rc 文件，那么就是进入了 Zygote 进程的 main 函数里了
app_main.cpp#main
- AndroidRuntime.cpp#start "com.android.internal.os.ZygoteInit"
-- startVm 启动 Java 虚拟机
-- startReg 注册 JNI 方法
-- toSlashClassName 替换 . 为 /
-- FindClass 找到 ZygoteInit.java 类
-- GetStaticMethodID 得到 ZygoteInit#main 方法
-- CallStaticVoidMethod 通过 JNI 方式调用该 main 方法
接着进入到 ZygoteInit#main 方法里
- ZygoteServer#registerServerSocket 创建名为"zygote"的 Server Socket
- ZygoteInit#preload 预加载类和资源
- ZygoteInit#startSystemServer，对应类名 "com.android.server.SystemServer"
- ZygoteServer#runSelectLoop 等待 AMS 请求创建新的应用程序进程
```



Zygote 进程

- Android.mk 定义名字 "app_process "，进程启动后，Linux 系统下的 pctrl 系统调用 app_process ，将名字改成 "zygote"
- init 进程启动时创建了 Zygote 进程
- Zygote 进程启动时会创建 ART（Dalvik）
- 通过 AndroidRuntime.cpp#start 函数启动 Zygote 进程(ZygoteInit)
- 创建 Java 虚拟机，并为 Java 虚拟机注册 JNI 方法
- 通过 JNI 调用 ZygoteInit#main 方法，ZygoteInit#main 是 Java 框架层面的入口
- 利用 registerServerSocket 创建 Server Socket ，runSelectLoop 等待 AMS 请求，预加载类和资源
- 并按需利用 fork 自身创建应用程序进程和 SystemServer 进程
- ps：ZygoteInit#main 间接调用了 ActivityThread#main



###### SystemServer 进程启动流程

```
ZygoteInit#main
- ZygoteInit#startSystemServer
-- ZygoteInit#handleSystemServerProcess
--- ZygoteInit#createPathClassLoader
--- ZygoteInit#zygoteInit
---- ZygoteInit#nativeZygoteInit 通过调用 c 层代码在新进程中创建 Binder 线程池
----- AndroidRuntime.cpp 里函数继续调用 app_main.cpp#onZygoteInit 函数，然后是 app_main.cpp#startThreadPool 
---- RuntimeInit#applicationInit
----- RuntimeInit#invokeStaicMain 通过反射得到 SystemServer 类，对应类名 com.android.server.SystemServer，以及他的 main 方法，最后通过抛异常的方式间接调用了 SystemServer#main 方法
- new SystemServer().run()
-- Looper#prepareMainLooper 是不是和之前学的基础 ActivityThread#main 里的套路相识？
-- System.loadLibrary("android_services") 加载 so 库
-- createSystemContext
-- new SystemServiceManager
-- SystemServer#startBootstrapServices 引导服务，启动了 AMS、PowerManagerService
-- SystemServer#startCoreServices 核心服务，启动了 BatteryService
-- SystemServer#startOtherServices 其他服务，启动了 WMS、CameraService、PackageManagerService
三类服务启动方式类似，并且会在服务 List 的集合中 add 进去
SystemServiceManager#startService
-SystemService#onStart
ps：另外也可以通过各个服务自身的 main 方法实例化自己
```

SystemServer 进程

- 启动新进程的 Binder 线程池
- 创建系统服务管理类 SystemServiceManager

- 创建各种系统服务，如 AMS、WMS、PackageManagerService



###### -> Launcher 桌面启动器启动流程

Launcher 的入口是 AMS 的 systemReady 方法

```
接上文【SystemServer 进程启动流程】，SystemServer#startOtherServices 启动其他服务
内部会调用 ActivityManagerService#systemReady
接着一系列调用后到达 ActivityManagerService#startHomeActivityLocked
然后是 ActivityStarter#startHomeActivityLocked
最终是 Launcher#onCreate 方法
- LauncherModel#loadWorkspace
- LauncherModel#bindWorkspace
- LauncherModel#loadAllApps 加载所有已安装 APP 的信息
AllAppsContainerView#onFinisnInflate 
- 找到 id 为 apps_list_view 的 RecyclerView 把 apps 列表的数据显示出来
```



###### 应用程序进程启动流程

上文【Zygote 进程启动流程】Zygote 进程的 Server Socket 等待 AMS 请求创建新的应用程序进程

应用程序启动的条件是应用程序进程是否已经启动，而 AMS 在启动应用程序的时候会先检查这个应用程序需要的应用程序进程是否已经启动，如未启动则会请求 Zygote 进程去先启动这个应用程序进程

```
=================== AMS 发送请求到 Zygote ==================
AMS 利用 ActivityManagerService#startProcessLocked 方法通过建立 Socket 连接发送请求给 Zygote 进程，提供一个app 的uid 和 entryPoint="android.app.ActivityThread"，他就是应用程序进程主线程的类名
- Process#start 尝试启动进程
-- 调用 ZygoteProcess#start
--- ZygoteProcess#startViaZygote
=================== Zygote 处理 AMS 请求 ==================
Zygote 进程在 ZygoteServer#runSelectLoop 方法下等待 AMS 的请求
然后利用 Zygote#forkAndSpecialize 方法 fork 方式创建应用程序进程
接着调用 ZygoteConnection#handeChildProc
- ZygoteInit#zygoteInit 接下来就和上文【SystemServer 进程启动流程】中段流程基本一致了
-- ZygoteInit#nativeZygoteInit 通过调用 c 层代码在新进程中创建 Binder 线程池
--- AndroidRuntime.cpp 里函数继续调用 app_main.cpp#onZygoteInit 函数，然后是 app_main.cpp#startThreadPool 
-- RuntimeInit#applicationInit
--- RuntimeInit#invokeStaicMain 通过反射得到 ActivityThread 类，对应类名 android.app.ActivityThread，以及他的 main 方法，最后通过抛异常的方式间接调用了 ActivityThread#main 方法
- Looper#prepareMainLooper 上文【SystemServer 进程启动流程】中提到的问号
- thread = new ActivityThread 主线程管理类
- sMainThreadHandler = thread.getHandler 用于处理主线程的消息循环
- Looper#loop
```

应用程序进程

- 启动新进程的 Binder 线程池
- 创建主线程管理类 ActivityThread
- 创建一个用于处理主线程的消息循环



### App 启动流程(根 Activity 启动流程)

```
在 Launcher 中点击 App 图标启动应用
- Launcher#startActivitySafely 
-- Activity#startActivity 带上 FLAG_ACTIVITY_NEW_TASK 标记，基础知识
--- Activity#startActivityForResult
---- Instrumentation#execStartActivity
----- ActivityManager.getService().startActivity 利用 AIDL 进程间通信方式与 AMS 通信，最终就是调用到 ActivityManagerService#startActivity
然后后面经过一系列调用会到 ActivityStarter#startActivityLocked 是不是和 Launcher 中间一段类似？
- ActivityStarter#startActivity 
- ActivityManagerService#getRecordForAppLocked 得到 Launcher 的 ProcessRecord
- new ActivityRecord
- 这个 ProcessRecord 就是需要传入的要启动 Activity 所在的应用程序进程
走到后面如果这个 ProcessRecord 为 null，表示应用程序进程未启动，那么走 ActivityManagerService#startProcessLocked 通知 Zygote 去启动该应用程序进程，上文【应用程序进程启动流程】提到的知识得到印证了
ProcessRecord 不为空的话经过一系列调用会到 ActivityThread$ApplicationThread.thread.scheduleLaunchActivity

小结一下：
1 Launcher 点击， 通过 AIDL ，Binder 通信方式到 AMS
2 而 AMS 是在 SystemServer 进程中的
3 AMS 想调用 ActivityThread$ApplicationThread 的方法
4 如果 ActivityThread$ApplicationThread 所在进程（应用程序进程）未启动，AMS 通过 socket 方式通知 Zygote 进程去完成对应进程的创建
5 如果已经创建了，则 AMS 通过 Binder 方式和 ActivityThread$ApplicationThread 所在进程进行通信，请求启动根 Activity

回到 scheduleLaunchActivity 继续
ActivityThread#sendMessage 通过 Handler 调用 H#sendMessage 方式发送通知启动对应 Activity
通过 Handler 的回调 H#handleMessage 方法处理调用 ActivityThread#handleLaunchActiviy 接下来的逻辑就比较基础了，是不是很熟悉？
```

启动流程涉及到 4 个进程：
Launcher 进程点图标请求 AMS 所在进程 SystemServer 进程请求启动应用程序根 Activity
AMS 判断该应用程序的应用程序进程是否已经启动
如果未启动则 AMS 通知请求 Zygote 进程先去创建并启动这个应用程序进程
应用程序进程启动后的逻辑就是 AMS 通知应用程序进程里的 ActivityThread$ApplicationThread 启动根 Activity





### Android 资源加载机制








### AsyncTask 异步任务

- 内部通过 Handler 实现






 
### RecyclerView 和 ListView 的区别
- RecyclerView 原生支持纵向、横向和瀑布流等布局，ListView 只支持纵向布局
- RecyclerView 利用 ItemDecoration 实现分割线，较灵活，ListView 利用 android:divider
- RecyclerView 自带 ViewHolder 机制，ListView 不强制要求实现 ViewHolder 机制
- RecyclerView 自带 Item 动画效果，ListView  没有
- RecyclerView 自带局部刷新
- RecyclerView 自带拖拽、侧滑接口
- RecyclerView 未实现头尾视图、空视图和 Item 点击事件
- RecyclerView 改进了缓存机制，有四级缓存
- RecyclerView 可扩展性强


 
### DiffUtil
- 实现 DiffUtil#Callback 定义新旧数据以及 Item 数据的比较规则
- 通过 DiffUtil#calculateDiff 计算得到 DiffResult （推荐放在子线程）
- 利用 DiffResult#dispatchUpdatesTo 通知刷新

注意：1 需要解决 Adapter 与 DiffUtil 的数据要一致性问题，每次刷新需要同时设置 Adapter 和 DiffUtil 的数据

​			2 定义维护新旧数据的时候需要注意引用传递问题



### 已经有 Adapter#notifyItemXXX 局部刷新了为啥还要有 DiffUtil 工具类？
- DiffUtil 最终也是调用 Adapter#notifyItemXXX 局部刷新方法进行刷新，只是工具类帮你计算了，不需要自己去计算维护哪些 position 需要刷新
- 如果只改变了个别 Item ，那么直接调用 Adapter#notifyItemXXX 即可，那就没必要用 DiffUtil 了
- DiffUtil 设计不单单是只能和 RecyclerView 一起使用，只要实现 ListUpdateCallback 接口的都可以用它来计算新旧数据集差异




### AsyncListDiff
- 通过 XxxAdapter 封装 AsyncListDiffer 进行使用，内部实现在子线程中执行 DiffUtil#calculateDiff  方法
- 实现 DiffUtil#ItemCallback 定义 Item 数据的比较规则，内部自行维护了新旧数据集
- 利用 AsyncListDiff#submitList 进行数据更新，同时自动完成刷新



### 已经有 DiffUtil  了为啥还要 AsyncListDiff ？
- 无需自行维护新旧数据集，解决了 Adapter 与 DiffUtil 的数据引用的一致性问题和数据引用传递问题
- 内部已经实现了通过子线程进行差异计算，解决了主线程阻塞问题
- 调用数据更新的时候会自动刷新接口，避免遗忘





### androidx.recyclerview.widget.ListAdapter
- 可以让 XxxAdapter 直接继承 ListAdapter ，简化了 AsyncListDiff 【通过 XxxAdapter 封装 AsyncListDiffer 进行使用】这一步骤

按顺序选型：
Adapter#notifyDataSetChanged -> 尽量选用局部刷新 Adapter#notifyItemXXX -> 数据集变化较复杂就用 DiffUtil 配合处理 -> 数据集较大可以选用 AsyncListDiffer 处理 -> 不想封装 Adapter 就直接继承 ListAdapter 处理


 

## Retrofit 

### 请求接口有多个 BaseUrl 的话要怎么处理？

-  最基础的就是用不同的 baseUrl 新建不同的 Retrofit 实例




 
 

 

## Material

ShapeableImageView
- 自带，可实现圆角等效果





## 性能优化

### 绘制优化

FPS Frames Per Second

肉眼在 60fps 以上就不会感觉卡顿

1/60=16.667



Profile GPU Rendering

通过彩色柱状图来表示出渲染问题，可在设备开发者选项里打开



显示 GPU 过渡绘制

通过彩色线条来表示多度绘制，可在设备开发者选项里打开



Systrace

数据采样分析工具，可在 DDMS 中打开使用，也可以在代码中编写 Trace 指定语句使用



TraceView

SDK 中自带的数据采集分析工具，可在 DDMS 中打开使用，也可以在代码中编写 Debug 指定语句使用



Hierarchy Viewer

SDK 自带可视化布局调试工具



Android Lint

代码扫描工具



<include/>、<merge/>、<ViewSub/>

include 对引用优化，可以把通用布局抽象出来

merge 合并布局，一般和 include 一起使用

ViewSub 通过 android:layout 指定待加载布局文件，java code 利用 inflate 触发加载，不过只能加载一次，不能实现显示与隐藏切换，不能嵌套<merge/>，ViewSub 操作的是布局



 

### 如何缩减APK包大小？





1 Memory Monitor

观察内存使用情况

2 Allocation Tracker

跟踪内存分配情况

3 Heap Dump

观察内存使用情况

4 MAT  

Memory Analysis Tool

分析堆存储文件 hprof，DDMS 或者 Memory Monitor 生成

5 Dominator Tree

分析对象实例引用关系

6 Histogram

分析类的引用关系

7 OQL

Object Query Language 

对象查询语言，用于查询当前内存中满足指定条件的所有对象

8 LeakCanary

基于 MAT ，在代码加入监测内存泄漏语句



#### Fragment如果在Adapter中使用应该如何解耦？

#### 设计一个音乐播放界面，你会如何实现，用到那些类，如何设计，如何定义接口，如何与后台交互，如何缓存与下载，如何优化？

#### 从0设计一款App整体架构，如何去做？

#### 项目框架里有没有Base类，BaseActivity和BaseFragment这种封装导致的问题，以及解决方法？


#### 实现一个库，完成日志的实时上报和延迟上报两种功能，该从哪些方面考虑？





安全问题
Android与服务器交互的方式中的对称加密和非对称加密是什么?
对于Android 的安全问题，你知道多少






协程构建器


viewLifecycleOwner.lifecycleScope 绑定fragment 的onCreateView()到 onDestroyView()这个范围的生命周期


lifecycleScope 绑定 fragment 的整个生命周期onCreate()到onDestroy()这个范围的生命周期，生命周期范围会更长



//get方法通过operator可以简写为[Key]

 //plus方法通过operator可以简写为 + 



 CPS 变换使同步代码异步化，增加额外的 Continuation 类型的参数，用于函数结果值的返回



 https://www.cnblogs.com/jingdongkeji/p/17784848.html#:~:text=%E7%BB%B4%E5%9F%BA%E7%99%BE%E7%A7%91%EF%BC%9A%E5%8D%8F%E7%A8%8B%EF%BC%8C%E8%8B%B1%E6%96%87Coroutine%20%5Bk%C9%99ru%E2%80%99tin%5D%20%EF%BC%88%E5%8F%AF%E5%85%A5%E5%8E%85%EF%BC%89%EF%BC%8C%E6%98%AF%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A8%8B%E5%BA%8F%E7%9A%84%E4%B8%80%E7%B1%BB%E7%BB%84%E4%BB%B6%EF%BC%8C%E6%8E%A8%E5%B9%BF%E4%BA%86%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%AD%90%E7%A8%8B%E5%BA%8F%EF%BC%8C%E5%85%81%E8%AE%B8%E6%89%A7%E8%A1%8C%E8%A2%AB%E6%8C%82%E8%B5%B7%E4%B8%8E%E8%A2%AB%E6%81%A2%E5%A4%8D%E3%80%82,%E4%BD%9C%E4%B8%BAGoogle%E9%92%A6%E5%AE%9A%E7%9A%84Android%E5%BC%80%E5%8F%91%E9%A6%96%E9%80%89%E8%AF%AD%E8%A8%80Kotlin%EF%BC%8C%E5%8D%8F%E7%A8%8B%E5%B9%B6%E4%B8%8D%E6%98%AF%20Kotlin%20%E6%8F%90%E5%87%BA%E6%9D%A5%E7%9A%84%E6%96%B0%E6%A6%82%E5%BF%B5%EF%BC%8C%E7%9B%AE%E5%89%8D%E6%9C%89%E5%8D%8F%E7%A8%8B%E6%A6%82%E5%BF%B5%E7%9A%84%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80%E6%9C%89Lua%E8%AF%AD%E8%A8%80%E3%80%81Python%E8%AF%AD%E8%A8%80%E3%80%81Go%E8%AF%AD%E8%A8%80%E3%80%81C%E8%AF%AD%E8%A8%80%E7%AD%89%EF%BC%8C%E5%AE%83%E5%8F%AA%E6%98%AF%E4%B8%80%E7%A7%8D%E7%BC%96%E7%A8%8B%E6%80%9D%E6%83%B3%EF%BC%8C%E4%B8%8D%E5%B1%80%E9%99%90%E4%BA%8E%E7%89%B9%E5%AE%9A%E7%9A%84%E8%AF%AD%E8%A8%80%E3%80%82


# 结构化并发
 协程结构背后的机制称为结构化并发。与全局范围相比，它具有以下优势：

作用域通常负责子协程，其生命周期与作用域的生命周期相关联。
如果出现问题或用户改变主意并决定撤销操作，该范围可以自动取消子协程。
该范围会自动等待所有子协程完成。因此，如果 scope 对应于一个协程，则父协程在其 scope 中启动的所有协程都已完成之前不会完成。





有2种方式可以等待一个协程执行完成：
调用launch时会返回一个job实例，调用job的join方法
调用async时会返回一个Deferred(Job的一种类型)，调用Deferred的await方法

2.为什么在该代码中要写入join呢，如果不写这句话 不会输出GlobalScope里的内容 因为GlobalScope有自己的作用域


可以使用 join() 等待启动的协程完成，但它不会传播其异常。但是，崩溃的子协程也会取消其父协程，并出现相应的异常。


cancelAndJoin方法会取消协程并等待它完成。

invokeOnCompletion 
invokeOnCancellation

suspend fun test4() = suspendCancellableCoroutine<String> { cont ->
    val call = OkHttpClient().newCall(...)
    // 定义一个取消回调事件
    cont.invokeOnCancellation {
        // 取消请求
        call.cancel()
    }
}

Thread.join方法用于等待另一个线程完成。比如在主线程中调用t.join（t是另一个线程），主线程会阻塞，直到线程t执行完毕。而yield方法主要是关于当前线程主动让出 CPU 资源，与等待其他线程完成没有关系。

而join主要就是干了一件事，即等待设置了isActive=false的协程执行完成

java 的 yield


结构化并发

需要注意的是，yield 并不能保证一定会立即让出 CPU 资源。
它只是向调度器发出了一个信号，表示当前线程愿意暂时让出 CPU 资源。调度器可能会忽略这个信号，让当前线程继续执行。
当有多个相同优先级的线程在竞争 CPU 资源，并且希望每个线程都能有比较公平的执行机会时，可以使用yield。例如，在一个简单的多线程模拟程序中，多个线程执行相似的任务，为了避免某个线程长时间占用 CPU 而导致其他线程饥饿（得不到执行机会），可以适时地调用yield方法。不过，在实际的复杂应用中，由于线程调度的不确定性，yield的使用效果可能很难准确预估，所以需要谨慎使用






Semaphore 
Phaser 



Fragment 作为 LifecycleOwner 的问题
无法保证 Activity/Fragment 停止后不继续执行启动，通过 Lifecycle 得到的状态来进行先判断再执行逻辑，能够规避该问题



所以，只要没有入侵更改版本号的，基本上都还是粘性事件，只是通过某种手段进行过滤。
- LiveData 即使没有活跃的观察者的情况下也会保持数据，当有新的观察者注册或者生命周期状态从非活跃变为活跃时，观察者会立即接收到最新的数据





转换LiveData：可以使用 Transformations 类对 LiveData 进行转换。
合并LiveData：可以使用 MediatorLiveData 来合并多个 LiveData 源。

通过 Transformations.map() 和 Transformations.switchMap() 方法，可以从一个 LiveData 转换为另一个 LiveData。

MediatorLiveData：这是一种特殊的 LiveData 类，它可以观察其他 LiveData 对象并做出反应。这允许你合并多个 LiveData 数据源。



1. 为什么Fragment中要使用viewLifecycleOwner代替this
在Android开发中，Fragment与Fragment中的View的生命周期并不一致。
我们在使用一些可观察的数据（比如LiveData）时，需要让观察者（observer）准确感知Fragment中的View的生命周期，而不是Fragment本身的生命周期。
基于这样的需求，Android专门构造了与Fragment中的View相对应的LifecycleOwner，也就是viewLifecycleOwner。以下是相关代码示例说明其重要性：










单例模式：双重校验锁，volatile关键字 

- ### volatile 关键字

  1. 保证可见性,不保证原子性

  2. 禁止指令重排序

  3. 不缓存,每次都是从主存中取

     https://www.cnblogs.com/zhengbin/p/5654805.html

  ### hashmap 原理



### 红黑树



二叉树



### jvm内存



- Java堆





### 类加载器

Android平台上虚拟机运行的是Dex字节码,一种对class文件优化的产物,传统Class文件是一个Java源码文件会生成一个.class文件，而Android是把所有Class文件进行合并，优化，然后生成一个最终的class.dex,目的是把不同class文件重复的东西只需保留一份,如果我们的Android应用不进行分dex处理,最后一个应用的apk只会有一个dex文件。
 Android中常用的有两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader。区别在于调用父类构造器时，DexClassLoader多传了一个optimizedDirectory参数，这个目录必须是内部存储路径，用来缓存系统创建的Dex文件。而PathClassLoader该参数为null，只能加载内部存储目录的Dex文件。所以我们可以用DexClassLoader去加载外部的apk。




 


### 线程池

- 对线程统一管理，统一调度的工具，可以复用已存在的线程，减少线程的创建和销毁的开销，降低消耗，增加速度

- 能够控制并发的线程数的上限，既能够降低资源的消耗浪费，也一定程度上避免了系统处理时候的堵塞，提高了系统资源的合理利用率，增强了对线程的管理性

- 能够实现定时周期任务

  ​	

### 内置线程池

FixedThreadPool
SingleThreadPool
ScheduleThreadPool
CachedThreadPool

### 自定义线程池





注解是什么，元注解又是什么？

注解：是针对代码的一种标注、描述、解释，可以针对代码来配置一些功能，能够用来自动生成 Java 代码



元注解：修饰注解的注解就叫元注解

- @Target
- @Retention
- @Inherited
- @Documented



### effectively final

- 事实上地、实际上地

java 8 后，匿名内部类、Lambda 表达式访问的局部变量不需要再显示声明成 final 了

[为什么必须是final的呢？ (cuipengfei.me)](https://cuipengfei.me/blog/2013/06/22/why-does-it-have-to-be-final/)





### effective java

- 有价值的，有效的


### Activity 启动流程

startActivity最终都会调用startActivityForResult，通过ActivityManagerProxy调用system_server进程中ActivityManagerService的startActvity方法，如果需要启动的Activity所在进程未启动，则调用Zygote孵化应用进程，进程创建后会调用应用的ActivityThread的main方法，main方法调用attach方法将应用进程绑定到ActivityManagerService（保存应用的ApplicationThread的代理对象）并开启loop循环接收消息。ActivityManagerService通过ApplicationThread的代理发送Message通知启动Activity，ActivityThread内部Handler处理handleLaunchActivity，依次调用performLaunchActivity，handleResumeActivity（即activity的onCreate，onStart，onResume）

 

### Android 操作系统启动流程

 
### App 启动流程

 
  

## 浏览器中输入一个 URL  然后回车访问，这个过程中间发生了什么？

- DNS 解析，把网址解析找到域名对应的 IP 地址
- 客户端发起 HTTP 请求，会进行 TCP 三次握手建立连接，服务端发送报文给客户端
- 客户端收到报文解析得到 HTML页面等数据，然后构建 DOM 树，再构造呈现树并绘制到浏览器页面上面



 

前后台传输数据需要用密钥对数据加密，那加密过程应该放在哪个位置


 
 