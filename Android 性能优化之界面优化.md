# Android 性能优化之界面优化
- FrameTime 两帧画面间隔耗时（也可简单认为是单帧渲染耗时）
- Frame Rate 帧率：指的是 1 秒钟播放多少帧
- FPS 每秒帧数（Frames Per Second）：就是 1 秒内画面刷新次数，其实 FPS 就是帧率的简称
- Frame Dropped 掉帧：因性能不足（渲染耗时过长）或网络延迟等问题导致实际帧率低于目标预期值，无法按目标时间绘制渲染完所有的帧，从而导致部分帧被丢弃了，表现为界面卡顿或画面不流畅
停滞
- Frame Lost 丢帧（Frame Discard 抛帧、 Frame Throttled 压帧）：主动丢帧，由于性能限制，主动地丢弃一些帧以保持画面的流畅性，可能帧率不及预期，但能够满足界面不卡顿
- Frame Skipped 跳帧：主动有策略地跳过某些帧的显示（可能是连续或非关键帧），但可能帧率能够维持预期值（比如视频倍速播放）地
- Jank 卡顿次数（精确卡顿率）：帧率波动、画面不连续，卡顿通常是由掉帧引起的

## 优化策略
- 在 onDraw 方法中避免创建新对象，减少内存分配和垃圾回收
- 减少布局层级嵌套
    - 优先使用 ConstraintLayout 约束布局、RelativeLayout相对布局等减少布局嵌套
    - 使用 `<include>` 标签复用布局，使用 `<merge>` 和 `<ViewStub>` 标签优化布局层级
- 通过打开开发者选项 -> 调试 GPU 过度绘制，通过显示的不同颜色来区分是存在过度绘制
- 通过 Tools -> Layout Inspector 布局检查器工具查看布局层级，排查是否存在多层无用的嵌套
- 利用 dumpsys 系统抓取器辅助分析卡顿原因，通过 adb shell dumpsys gfxinfo <package_name> 获取系统界面性能状态信息，进行排查优化
- Systrace 系统跟踪工具是一个强大的系统级性能数据采样和分析工具，记录设备一段时间内的用户交互、CPU 活动和系统事件等信息数据，可用于识别应用中的卡顿，通过执行命令生成 HTML 报告供查看分析，也可以在代码中使用 Trace#traceBegin 和 Trace#traceEnd 记录应用内自定义日志
- 利用 Android Studio Profiler 分析应用性能，排查问题原因

## Looper 消息循环日志监控
- 替换主线程 Looper 的 Printer 日志打印，通过在消息执行前后的打印信息计算消息执行时间，从而判断是否存在卡顿，若 dispatchMessage 执行时间异常，则判定为卡顿
```java
Looper.getMainLooper().setMessageLogging(new Printer() {
    @Override
    public void println(String x) {
        //
        if (x.startsWith(">>>>>")) { //消息开始
            startTime = System.currentTimeMillis();
        } else if (x.startsWith("<<<<<")) { //消息结束
            long duration = System.currentTimeMillis() - startTime;
            //判断
        }
    }
});
```

## Choreographer 编舞者进行帧监控
- 进行掉帧检测，在每一帧回调时计算帧间隔时间，若间隔时间超过 16 毫秒，就认为可能存在掉帧的情况
```java
public class FrameMonitor implements Choreographer.FrameCallback {
    //系统每 16.67ms（60Hz 每 1 秒刷新 60 次，即 1000/60）发出一个 VSYNC 信号来通知刷新一次屏幕
    //假设在界面不卡顿的情况下，界面应该至少间隔 16.67ms 刷新一次，因此理论上至少每 16.67ms 应该会触发一次回调
    //如果发生界面卡顿了，那么回调的触发时间间隔就会超过 16.67 ms
    private static final long FRAME_TIME_THRESHOLD = 16L; //毫秒
    private long lastFrameTimeNanos = 0;

    @Override
    public void doFrame(long frameTimeNanos) {
        if (lastFrameTimeNanos != 0) {
            long jitterMillis = (frameTimeNanos - lastFrameTimeNanos) / 1000000;
            if (jitterMillis > FRAME_TIME_THRESHOLD) {
                //计算相邻帧时间差
                System.out.println("帧间隔过长: " + jitterMillis + " ms");
            }
        }
        lastFrameTimeNanos = frameTimeNanos;
        //注册下一帧回调
        Choreographer.getInstance().postFrameCallback(this);
    }

    public void start() {
        Choreographer.getInstance().postFrameCallback(this);
    }

    public void stop() {
        Choreographer.getInstance().removeFrameCallback(this);
    }
}    
```

## FrameMetrics
- Android 高版本直接获取帧率相关数据信息
```java
public class MainActivity extends Activity {
    private Window.OnFrameMetricsAvailableListener frameMetricsListener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //创建帧指标监听器
        frameMetricsListener = new Window.OnFrameMetricsAvailableListener() {
            @Override
            public void onFrameMetricsAvailable(Window window, FrameMetrics frameMetrics, int dropCountSinceLastInvocation) {
                // 
                //表示完整帧（获取到 Vsync 信号到帧完成的总时间）的时间
                long totalDuration = frameMetrics.getMetric(FrameMetrics.TOTAL_DURATION);
                //处理帧指标数据，可以在这里进行日志记录或其他操作
            }
        };
        //添加监听器
        getWindow().addOnFrameMetricsAvailableListener(frameMetricsListener, null); //可以指定 Handler
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //移除监听器
        if (frameMetricsListener != null) {
            getWindow().removeOnFrameMetricsAvailableListener(frameMetricsListener);
        }
    }
}
```

## BlockCanary
- 基于 Looper 的 Printer 日志打印，内部也是使用了 Looper.getMainLooper().setMessageLogging 通过替换主线程 Looper 打印的日志方案实现

## 腾讯 Matrix
- 结合了 Choreographer 和 Looper 监控方案，分析 UI 线程的 Message 执行耗时

## JankStats 卡顿统计（Metrics 指标库）
- Android 高版本采用 FrameMetrics，Android 低版本采用 Choreographer
```groovy
dependencies {
    implementation "androidx.metrics:metrics-performance:1.0.0-beta02"
}
```
 