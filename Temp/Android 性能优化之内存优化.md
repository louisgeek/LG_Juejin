# Android 性能优化之内存优化

## GC 垃圾回收机制
- Gabage Collection 垃圾收集
- 垃圾回收是一种自动的存储管理机制。 当一些被占用的内存不再需要时，就应该予以释放，以让出空间
- 在 JVM 中，对于对象的回收 GC 是基于**可达性分析**。简单来说，就是从 GC Root 出发，被引用的对象均被标记为存活，而没有被引用的对象，则被标记为垃圾，即可以被 GC 回收。

## 分析工具
- Android Studio Profiler
- Square LeakCanary Square 提供的内存泄漏检测框架库 
Traceview
Systrace
Strictmode
Matrix

## 内存泄漏
- 静态变量持有生命周期短的对象
- 非静态内部类或匿名内部类
- 子线程耗时操作运行未结束
- Handler 相关的内存泄漏
- 异步处理耗时任务，在finish时未终止(new Thread)
- 资源对象（比如 Cursor、Bitmap）未关闭或者释放

## OOM 内存溢出
- Out Of Memory 内存不足、内存溢出：指程序在申请内存时，当下应用内存没有足够的内存空间供分配使用，则会导致程序崩溃或异常终止的现象，也就是出现了内存溢出
- 比如出现一定量的内存泄漏，随着内存泄露的增长而导致内存占用持续不断增加，最终导致 OOM
- 大量使用 Bitmap 导致的 OOM
- 内存中存在大量的强引用对象
 
### 解决
用 StringBuilder、StringBuffer 代替 String
用 ArrayMap、SparseArray 替换 HashMap
3.内存对象的重复利用；
1.适合选择不同的引用方式创建对象（强、软、弱、虚）；

## 内存抖动
在短时间内有大量的对象被创建或者被回收的现象。（**循环种大量创建对象、回收内存GC**）
短时间内有大量的对象被创建和回收。类比生活中的正弦函数，在半个周期内，我们不断的创建对象，申请内存，很快就攀到了波峰，接着出现了GC(Stop The World)，清理内存，几乎一瞬间就到了起步线。这样一个急上急下的折线图，就是内存抖动的表现。

Android的UI卡顿就是内存抖动导致，移步了解UI卡顿与内存抖动的前世今生
 



## Android Studio Profiler
- Profiler 分析器，用于实时分析和监测 Android 应用程序性能的工具，可以帮助开发者识别和解决各种性能问题，包括 CPU 使用情况、内存分配、网络请求活动和电量消耗等
- Heap Dump 堆转储：捕获应用程序的内存快照，分析对象的分配和内存泄漏情况
- Record Allocation：跟踪记录对象的创建和销毁
- View -> Tool Windows -> Profiler
- Home
    - 进程名 PID 等
    - Tasks
- Past Recordings 之前的 Task 记录
    - Recorded tasks 记录的 Task 的类型
    
- Table 表格
- Visualization 可视化

### Analyze Memory Usage 分析内存使用情况（Heap Dump 堆转储）
- 捕获堆转储时 App 中正在使用内存的对象
- 在结果页面会显示存在的 Leaks，双击后可以选择对应的 Class -> Instance -> 可以查看 Fields 和 References，可以看到内存泄漏类的具体引用路径

第一行
```
View all heaps：查看所有
View app heap：查看 App 在其中分配内存的主堆
View image heap：系统启动映像，包含启动期间预加载的类，此处的分配确保绝不会移动或消失
View zygote heap：写时复制堆，其中的应用进程是从 Android 系统中派生的

Arrange by class：默认根据类名对所有分配进行分组
Arrange by package：根据软件包名对所有分配进行分组
Arrange by callstack：将所有分配分组到其对应的调用堆栈

Show all classes：默认展示所有 Class 类（包括系统类）
Show activity/fragment Leaks：展示泄露的 Activity/Fragment
Show project class：展示项目的 Class 类
```

第二行
```
Classes：类数量
Leaks：展示泄露数量，双击会自动选中 Show activity/fragment Leaks
Count：？
Native Size：此对象类型使用的原生内存总量（单位字节），只有在使用 Android 7.0 及更高版本时，才会看到此列，会在此处看到采用 Java 分配的某些对象的内存，因为 Android 对某些框架类（如 Bitmap）使用原生内存
Shallow Size：此对象类型使用的 Java 内存总量（单位字节）
Retained Size：为此类的所有实例而保留的内存总大小（单位字节）
```

第三行
```
Class Name：类名
Allocations：堆中的分配数，对象个数
Native Size
Shallow Size
Retained Size
```

第四行
```
//选中一个类名查看 Instance List，选中一个实例可在右侧打开 Instance Details 详情窗口查看 Fields 和 References
Depth：从任意 GC 根到选定实例的最短跳数
Native Size
Shallow Size
Retained Size
//
References 窗口有个 Show nearest GC root only 复选框
```

### Track Memory Consumption 跟踪内存消耗（Java/Kotlin Allocations 分配量）
- 录制 Java 和 Kotlin 内存分配情况

### Track Memory Consumption 跟踪内存消耗（Native Allocations 分配量）
- 录制本地内存分配情况




## Square LeakCanary
- Square 提供的内存泄漏检测框架库
- 老版本通常通过自定义 Application ，在 onCreate 方法中通过 LeakCanary.install 进行初始化
- 新版本不需要手动初始化，库里的 xml 注册的 ContentProvier（leakcanary.internal.AppWatcherInstaller 在 onCreate 方法中调 leakcanary.AppWatcher.manualInstall） 会自动初始化 LeakCanary
- 通过 application.registerActivityLifecycleCallbacks 来绑定 Activity 的生命周期监听，从而监控所有 Activity
- 根据要回收对象创建 KeyedWeakReference 并关联 ReferenceQueue，在弱引用关联的对象被回收后，会将引用添加到 ReferenceQueue 中，可以根据有没有该引用来判定是否被回收了，如果有这个对象则说明被回收了，否则说明没有被回收






