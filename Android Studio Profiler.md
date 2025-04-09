# Android Studio Profiler
- Profiler 分析器，用于实时分析和监测 App 性能的工具，可以帮助开发者识别和解决各种性能问题，包括 CPU 使用情况、内存分配、网络请求活动和电量消耗等
- Heap Dump 堆转储：捕获 App 当前内存的快照，分析对象的分配和内存泄漏情况
- Record Allocations：跟踪记录内存分配情况（对象的创建和销毁等）
- 从 View -> Tool Windows -> Profiler 打开，Profiler 界面分两个 Tab，分别是 Home 主页和 Past Recordings 之前的 Task 记录页
- Home 主页左侧显示设备和应用进程列表（展示进程名、PID 和 Manifest Configuration），右侧显示 Tasks 列表
- Past Recordings 之前的 Task 记录页显示之前的 Task（展示 Recording name、Recording time 和 Recorded tasks 记录的 Task 的类型）
```java
Tasks 列表
-Capture System Activities 捕获系统活动（System Trace 系统跟踪）
-Analyze Memory Usage 分析内存使用（Heap Dump 堆转储）
-Find CPU HotSpots（CallStack Sample 调用栈采样）
-Find CPU HotSpots（Java/Kotlin Method Recording 方法记录）
-Track Memory Consumption 跟踪内存消耗（Java/Kotlin Allocations 分配量）
-Track Memory Consumption 跟踪内存消耗（Native Allocations 分配量）
-View Live Telemetry 查看实时监测（View Live）
```

## Analyze Memory Usage 分析内存使用（Heap Dump 堆转储）
- 执行一次堆转储可以捕获当前的内存中所有对象的快照（保存生成一个 .hprof 堆转储文件），打开后可以查看不同类的对象数量、占用的内存大小等信息，分析寻找可能存在内存泄露的对象（特别注意那些不应该存在的 Activity、Fragment 等对象实例仍然占用着内存），查看这些对象的引用链，找出可能导致泄露的原因
- 在结果页面会显示存在的 Leaks，双击后可以选择对应的 Class -> Instance -> 可以查看 Fields 和 References，可以看到内存泄漏类的具体引用路径，References 窗口第一行有个 Show nearest GC root only 复选框

```java
//第一行
View all heaps：查看所有
View app heap：查看 App 在其中分配内存的主堆
View image heap：系统启动映像，包含启动期间预加载的类，此处的分配确保绝不会移动或消失
View zygote heap：写时复制堆，其中的应用进程是从 Android 系统中派生的
//
Arrange by class：默认根据类名对所有分配进行分组
Arrange by package：根据软件包名对所有分配进行分组
Arrange by callstack：将所有分配分组到其对应的调用堆栈
//
Show all classes：默认展示所有 Class 类（包括系统类）
Show activity/fragment Leaks：展示泄露的 Activity/Fragment
Show project class：展示项目的 Class 类
//第二行
Classes：类数量
Leaks：展示泄露数量，双击会自动选中 Show activity/fragment Leaks
Count：？
Native Size：此对象类型使用的原生内存总量（单位字节），只有在使用 Android 7.0 及更高版本时，才会看到此列，会在此处看到采用 Java 分配的某些对象的内存，因为 Android 对某些框架类（如 Bitmap）使用原生内存
Shallow Size：此对象类型使用的 Java 内存总量（单位字节）
Retained Size：为此类的所有实例而保留的内存总大小（单位字节）
//第三行
Class Name：类名
Allocations：堆中的分配数，对象个数
Native Size
Shallow Size
Retained Size
//第四行
//选中一个类名查看 Instance List，选中一个实例可在右侧打开 Instance Details 详情窗口查看 Fields 和 References
Depth：从任意 GC 根到选定实例的最短跳数
Native Size
Shallow Size
Retained Size
```

## Track Memory Consumption 跟踪内存消耗（Java/Kotlin、Native Allocations 分配量）
- Java/Kotlin Allocations 分配量，Java 和 Kotlin 内存分配情况
- Native Allocations 分配量，本地内存分配情况
- 支持光标拖动一个区间分析，可以点击停止图标按钮停止记录内存分配，可以点击垃圾桶图标按钮手动触发 GC 垃圾回收
- 查看观察内存使用情况实时变化的图表，模拟用户进行一些可能存在内存泄漏的操作（比如反复打开和关闭某个页面，让疑似泄漏的代码逻辑被执行），如果页面关闭后内存未下降，就可能存在内存泄露问题
- 可以按需手动触发 GC 垃圾回收来清理不再使用的对象，然后观察内存是否有明显的下降，如果内存没有明显下降，就可能存在内存泄漏问题
- 在结果页面可以选择对应的 Class -> Instance -> 可以查看 Allocation Call Stack 分配调用堆栈（可以右键选择 Jump to Source），另外除了 Table 表格，还可以通过 Visualization 可视化的方式呈现

## 总结
- 主要是通过捕获一次 Heap Dump 堆转储时正在使用内存的对象信息进行分析和通过 Record Allocations 记录分配量（Allocation Tracking）跟踪内存分配情况这两种方法进行交叉（结合）排查验证，分析确认以及解决内存泄露问题
- 需要特别关注那些实例数量不断增加或占用大量内存的对象，这些对象往往可能引发内存泄漏，查看分析对象的引用链，结合代码逻辑，确定哪些对象未被及时回收，找出可能导致泄露的原因
- 多次重复记录和分析过程，确认内存泄露是否已被修复
- 支持导出 Heap Dump（.hprof 文件），采用 MAT 工具（Memory Analyzer Tool）进行进一步的深度分析