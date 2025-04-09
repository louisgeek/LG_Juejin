# Android 性能优化之内存优化

## 内存泄漏
- 内存泄漏：指不再使用的对象因被引用而无法被回收，导致内存占用持续增加
- 1 减少静态对象的使用，需要及时进行清理（比如赋值为 null）释放引用
- 2 确保静态集合中的对象在不需要的时候能够被及时移除
- 3 使用 WeakReference 弱引用
- 4 减少单例类的使用，约束滥用行为
- 5 尽可能的改用 Application 的 Context
- 6 避免使用非静态内部类或匿名内部类
- 7 注意 Handler 导致的内存泄漏，及时在 onStop 或 onDestroy 时候用 removeCallbacksAndMessages 移除消息
- 8 针对线程相关及时做好取消、终止等工作（比如子线程耗时操作运行仍未结束）
- 9 针对资源性对象（比如数据库、File 文件和 Cursor 等）、WebView 和动画等需要及时做好回收关闭工作，做好 BroadcastReceiver 广播、监听器和观察者等的移除反注册工作，Service 服务执行完成后记得关闭
- 10 多使用 try–catch-finally、try-with-resources 特性或者 Kotlin 的 T.use 函数

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

## 检测分析
- LeakCanary 检测内存泄漏，定位泄漏原因
- 通过腾讯 Matrix​ 进行内存泄露检测（包括 native）、Bitmap 重复创建等
- 利用 Systrace​ 系统跟踪工具监控系统性能，识别问题原因
- 利用 Android Studio Profiler 分析应用性能，排查问题原因
- 导出 Heap Dump（.hprof 文件），采用 MAT 工具（Memory Analyzer Tool）进行进一步的深度分析堆转储文件，定位内存泄漏根源















