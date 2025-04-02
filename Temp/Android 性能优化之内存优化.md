# Android 性能优化之内存优化

## GC 垃圾回收机制
- Gabage Collection 垃圾收集
- 垃圾回收是一种自动的存储管理机制。 当一些被占用的内存不再需要时，就应该予以释放，以让出空间
- 在 JVM 中，对于对象的回收 GC 是基于**可达性分析**。简单来说，就是从 GC Root 出发，被引用的对象均被标记为存活，而没有被引用的对象，则被标记为垃圾，即可以被 GC 回收。

使用线程池处理异步任务
使用SparseArray代替HashMap存储键值对
LeakCanary检测内存泄漏
对于频繁创建和销毁的对象，使用对象池进行复用。
对于复杂的图像处理任务，使用RenderScript来加速计算
 使用DiffUtil进行增量更新
原理：DiffUtil通过比较新旧数据集差异，仅更新变化的Item，避免全局刷新
限制图片尺寸：避免加载原图
Systrace：分析UI线程和RenderThread的耗时Perfetto：跟踪CPU调度、锁竞争等系统级问题
开发者选项 → 显示过度绘制优化策略：移除不必要的背景使用canvas.clipRect()限制绘制区域


## 分析工具
- Android Studio Profiler
- Square LeakCanary 是 Square 提供的内存泄漏检测框架库 
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
动画也可能导致内存泄露，比如启动了属性动画(ObjectAnimator)，但是在销毁的时候，没有调用cancle方法，虽然我们看不到动画了，但是这个动画依然会不断地播放下去，动画引用所在的控件，所在的控件引用Activity，这就造成Activity无法正常释放。因此同样要在Activity销毁的时候cancel掉属性动画，避免发生内存泄漏。
 

## OOM 内存溢出
内存溢出，Android设备出厂以后，java虚拟机对单个应用的最大内存分配就确定下来了，超出这个值就会OOM，所以长期的内存泄露也会导致内存溢出，不过如果是一次性开辟太大的数组或者加载过大的文件图片也容易导致OOM
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
内存抖动，主要是内存波动图类似锯齿状，代表存在频繁地内存开辟和销毁，容易导致内存碎片，频繁的GC也容易导致卡顿
Android的UI卡顿就是内存抖动导致，移步了解UI卡顿与内存抖动的前世今生
 使用缓存池，减少对象频繁创建和销毁
减少不合理对象的创建，特别是嵌套for循环，注意对象创建的位置，尽量在for结构外侧。如果是Gson解析避免多处的重复创建。
使用合理的数据结构，SparseArray类，在清楚Map的个数的时候，可以手动设置大小个数，初始化个数都在16个，减少过多开辟



## Square LeakCanary
- Square 提供的内存泄漏检测框架库
- 老版本通常通过自定义 Application ，在 onCreate 方法中通过 LeakCanary.install 进行初始化
- 新版本不需要手动初始化，库里的 xml 注册的 ContentProvier（leakcanary.internal.AppWatcherInstaller 在 onCreate 方法中调 leakcanary.AppWatcher.manualInstall） 会自动初始化 LeakCanary
- 通过 application.registerActivityLifecycleCallbacks 来绑定 Activity 的生命周期监听，从而监控所有 Activity
- 根据要回收对象创建 KeyedWeakReference 并关联 ReferenceQueue，在弱引用关联的对象被回收后，会将引用添加到 ReferenceQueue 中，可以根据有没有该引用来判定是否被回收了，如果有这个对象则说明被回收了，否则说明没有被回收




用 StringBuilder、StringBuffer 代替 String

用 ArrayMap、SparseArray 替换 HashMap

避免内存泄漏

- 集合类泄漏(集合一直引用着被添加进来的元素对象)
- 单例/静态变量造成的内存泄漏(生命周期长的持有了生命周期短的引用)
- 匿名内部类/非静态内部类
- 资源未关闭造成的内存泄漏
  - 网络,文件等流忘记关闭
  - 手动注册广播时,退出时忘记unregisterReceiver()
  - Service执行完成后忘记stopSelf()
  - EventBus等观察者模式的框架忘记手动解除注册






