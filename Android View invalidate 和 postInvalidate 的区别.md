# Android View invalidate 和 postInvalidate 的区别
- 在 Android 中 View#invalidate 和 View#postInvalidate 都是用于请求系统去刷新视图重绘的方法，其中 invalidate 必须在 UI 主线程中调用，而 postInvalidate 可以用在非 UI 主线程中，postInvalidate 方法最后调用的也是 invalidate 方法
- 通常情况下在自定义 View 中修改了某些绘制相关的属性（如颜色、文本等）后，需要调用 invalidate 来告知系统这个视图需要重新绘制以反映这些属性的改变，系统会在下一个合适的时机进行一次重绘操作，此时会调用视图的 View#onDraw 方法，从而完成了视图实际的重绘操作
- 比如在后台线程中进行了一些数据处理等操作，并且这些数据需要反映到视图的绘制上时，就可以使用 postInvalidate 来触发视图重绘保证视图能够及时反映出数据的变化，而不需要手动去处理将重绘请求从后台线程转移到 UI 主线程的这样一个过程

## invalidate
- 当一个 View 调用 invalidate 方法时，它会将该视图所在的区域标记为无效状态，意味着当前视图所呈现的内容已不再准确反映其应有的状态，需要进行重绘以展示最新的状态，同时如果只是部分区域无效，那么就只重绘那部分区域，如果是整个视图无效，就重绘整个视图
- 调用 invalidate 后，View 视图并不是立即重绘的，而是会将重绘请求添加到当前线程的消息队列（即 Android UI 主线程的消息队列）中，然后在 UI 主线程的消息队列中，当这条重绘请求被处理时，此时系统才会调用视图视图的 View#onDraw 方法完成绘制逻辑
- 由于 invalidate 是一个在 UI 线程中调用的方法，如果在后台线程中调用 invalidate 方法会报异常，所以后台线程无法直接用 invalidate 实现刷新视图重绘的操作

```java
public void invalidate() {
    invalidate(true);
}
```

```java
@UnsupportedAppUsage
public void invalidate(boolean invalidateCache) {
     //invalidateCache 设置 View 的缓存是否失效，通常情况下是 true
     invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
}
```

```java
View#invalidate
-View#invalidateInternal
--ViewGroup#invalidateChild 实现的 ViewParent#invalidateChild 接口
---ViewRootImpl#invalidateChildInParent 实现的 ViewParent#invalidateChildInParent 接口
----ViewRootImpl#invalidateRectOnScreen 遍历到根布局后执行
-----ViewRootImpl#scheduleTraversals 内部有同步屏障逻辑
------mChoreographer.postCallback 设置了一个 mTraversalRunnable 对象，它是一个 Runnable，run 方法里就是执行 ViewRootImpl#doTraversal 方法
-------ViewRootImpl#doTraversal
--------ViewRootImpl#performTraversals 内部代码逻辑就是依次按照条件判断是否去执行 ViewRootImpl#performMeasure、ViewRootImpl#performLayout 和 ViewRootImpl#performDraw 方法，此处 performMeasure 和 performLayout 两个方法会因为条件不成立而跳过不执行，只有 performDraw 方法会执行，内部最终会调用到 View#draw 这个方法，然后 View#draw 方法就会调用 View#onDraw 方法进行视图重绘操作

PS：也就是说任意一个子 View 调用 invalidate 方法最终都会走到 ViewRootImpl 的 performTraversals 方法里；另外 invalidate 方法只会触发绘制这一流程而不会触发测量、布局这两大流程；Choreoprapher 类的作用是编排输入事件、动画事件和绘制事件的执行
```

## postInvalidate
- 而 postInvalidate 主要是为了解决在非 UI 主线程中无法直接调用 invalidate 的问题，方便在非 UI 主线程中能够方便顺利地触发视图重绘
- postInvalidate 其实就是通过内部的 Handler 消息机制将重绘请求消息发送到 UI 主线程的消息队列中来实现
- 内部的 postInvalidateDelayed 方法向 ViewRootImpl.ViewRootHandler 的 mHandler 对象发送一个延迟消息，而在这个 Handler#handleMessage 处理这个消息的地方实际上还是调用的 View的 invalidate 这个方法，因为这个 ViewRootImpl 是在 UI 主线程的，所以 postInvalidate 方法实际就是完成了将非 UI 主线程上的刷新视图操作切换到 UI 主线程上这么一个逻辑

```java
//View#postInvalidate
public void postInvalidate() {
    postInvalidateDelayed(0);
}
public void postInvalidateDelayed(long delayMilliseconds) {
    // We try only with the AttachInfo because there's no point in invalidating
    // if we are not attached to our window
    final AttachInfo attachInfo = mAttachInfo; //View 的静态内部类 AttachInfo，mAttachInfo 在 View#dispatchAttachedToWindow 方法里赋值
    if (attachInfo != null) {
        attachInfo.mViewRootImpl.dispatchInvalidateDelayed(this, delayMilliseconds);
    }
}
```

```java
//ViewRootImpl#dispatchInvalidateDelayed
public void dispatchInvalidateDelayed(View view, long delayMilliseconds) {
    //ViewRootImpl 的内部类 ViewRootHandler，mHandler 就是 ViewRootHandler 对象
    Message msg = mHandler.obtainMessage(MSG_INVALIDATE, view); //obj 存的就是这个 view
    //延迟消息
    mHandler.sendMessageDelayed(msg, delayMilliseconds);
}
```

```java
//ViewRootImpl.ViewRootHandler#handleMessage
@Override
public void handleMessage(Message msg) {
    if (Trace.isTagEnabled(Trace.TRACE_TAG_VIEW)) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, getMessageName(msg));
    }
    try {
        handleMessageImpl(msg);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
private void handleMessageImpl(Message msg) {
    switch (msg.what) {
        case MSG_INVALIDATE:
        ((View) msg.obj).invalidate(); //从 obj 取出这个 view，执行其 invalidate 方法
        break;
    }
}
```


