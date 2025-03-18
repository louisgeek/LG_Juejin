# Android Choreographer 编舞者
- 又称为舞蹈指导（舞蹈是有节奏的，节奏使舞蹈的每个动作更加协调和连贯，视图刷新亦是如此）
- VSync 就是 Vertical Synchronization 垂直同步
- Choreographer 主要负责统一协调管理 CALLBACK_INPUT 外部输入、CALLBACK_ANIMATION 动画和 CALLBACK_TRAVERSAL 遍历视图树（包括测量、布局和绘制）三大操作（新版还有一个 CALLBACK_COMMIT）的时机，当接收到来自系统的 VSync 垂直同步信号后，顺序执行输入、动画和遍历视图树这三个操作，然后等待下一个 VSync 信号到来后再次顺序执行这三个操作，实现控制这三个操作的同步处理，确保 UI 渲染能够在合适的时机进行，以实现流畅的动画和界面视图刷新
- 通过 Choreographer#postFrameCallback 设置一个 Choreographer.FrameCallback 回调，在每次屏幕刷新周期（即下一帧渲染）开始时（VSync 信号触发）调用 doFrame 方法，可以在回调里执行一些自定义代码（比如可以用来记录每一帧的时间、监测帧率或实现特定的自定义动画效果等）
- 监控 UI 渲染性能、卡顿检测排查：通过计算帧渲染的时间，分析帧间隔，定位卡顿原因，找出性能瓶颈
- 自定义动画效果：可以通过计算帧渲染的时间，按需进行动画速度、逻辑等的控制，因为 doFrame 方法回调节奏与屏幕刷新频率（VSync 信号）一致，如果在回调中更新动画的数值，就可以确保动画与屏幕刷新保持对齐，避免了过度渲染和无效计算，从而节省资源
- ViewRootImpl#scheduleTraversals 依赖 Choreographer 触发视图界面遍历，scheduleTraversals 方法内部会通过 Choreographer#postCallback 设置传入一个 Choreographer.CALLBACK_TRAVERSAL，另外 postCallback 方法和 postFrameCallback 方法最终都会调用到 postCallbackDelayedInternal 方法
- Choreographer 内部有一个 Looper 和一个 Handler（FrameHandler）
- Choreographer 内部也实现了对掉帧的监控逻辑，不过默认是监控超过 30 帧及以上打印逻辑

```java
Choreographer.getInstance().postFrameCallback(new Choreographer.FrameCallback() {
    //当新的一帧开始渲染时，doFrame 方法会被调用，不过应用在后台运行或者设备息屏的情况下，doFrame 方法回调将不会被触发
    @Override
    public void doFrame(long frameTimeNanos) { //frameTimeNanos 表示帧开始渲染时的时间，单位纳秒
        //如果回调在 UI 主线程上运行，需要避免进行耗时的操作
        //自定义代码逻辑
        //如果需要持续监听每一帧的事件，需要在 doFrame 方法中再次调用 postFrameCallback 方法继续监听下一帧回调
        Choreographer.getInstance().postFrameCallback(this);
    }
});


//需要在合适的时机移除回调，以防内存泄漏
Choreographer.getInstance().removeFrameCallback
```