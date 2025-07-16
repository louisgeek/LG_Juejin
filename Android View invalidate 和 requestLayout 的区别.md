
# Android View invalidate 和 requestLayout 的区别
- 在 Android 中 View#invalidate 和 View#requestLayout 都可以用于请求系统去刷新视图显示的方法，invalidate 方法会触发 draw 过程，而 requestLayout 方法则会触发测量、布局和绘制（可能还要走绘制阶段，比如视图的位置发生改变） 
- invalidate 和 requestLayout 方法最终都会调用到 ViewRootImpl#performTraversals 方法
- 通常情况下修改了某些绘制相关的属性（比如颜色、文本等）后，可以调用 invalidate 来通知系统，当前视图需要重新绘制反映其最新的状态
- 而修改了某些位置大小相关的属性（比如尺寸、位置等）后，就需要调用 requestLayout 来告知系统当前视图需要重新测量、布局以及绘制操作
- 在某些情况下可能需要同时调用这两个方法，先调用 requestLayout 来更新布局，然后调用 invalidate 来触发重绘（比如视图的位置未发生改变，但是绘制的属性需要改变，那就需要重新绘制）

## invalidate
- 侧重于当前视图的显示内容（比如颜色、绘制的图形、文本等外观）已经发生了变化，需要重新进行绘制（也就是触发 draw 过程），不涉及布局大小和位置等布局参数的重新确定

## requestLayout
- 在自定义视图中使用 requestLayout 时，会通知系统该视图的布局等信息已经改变，标记视图需要重新进行布局，之后在系统下一次绘制流程中会对视图进行测量、布局，如果视图的内容也发生了变化，那么还可能会触发绘制过程（requestLayout 不保证后续条件一定满足去重新绘制，所以某些情况下要进行重新绘制可以再按需手动调用 invalidate 方法）

```java
View#requestLayout
-ViewParent#requestLayout //因为 ViewRootImpl 是 DecorView 的 parent，所以最终会调用到 ViewRootImpl#requestLayout
--ViewRootImpl#requestLayout
---ViewRootImpl#scheduleTraversals //内部有 Handler 同步屏障逻辑 MessageQueue#postSyncBarrier
----Choreographer#postCallback //设置传入一个 mTraversalRunnable 对象，Runnable#run 方法里就是执行 ViewRootImpl#doTraversal 方法
-----ViewRootImpl#doTraversal
------ViewRootImpl#performTraversals //内部代码逻辑就是依次按照条件判断是否去执行 performMeasure、performLayout 和 performDraw 方法
-------ViewRootImpl#performMeasure
-------ViewRootImpl#performLayout
-------ViewRootImpl#performDraw
// ！！！！！！！！！
// ，此处 performMeasure 和 performLayout 两个方法会跳过不执行，而 performDraw 方法会执行，内部最终会调用到 View#draw 这个方法，然后 View#draw 方法就会调用 View#onDraw 方法进行视图重绘操作
```


```java
 //ViewRootImpl#performTraversals
 private void performTraversals() {
    //...
    //mStopped 可以代表 Activity 是否处于 Stopped 状态
    if (!mStopped || mReportNextDraw) {
        //...
        //测量
        // Ask host how big it wants to be
        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
        //...
    }

    //...
    final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
    //...
    if (didLayout) {
        //布局
        performLayout(lp, mWidth, mHeight);
        //...
    }

    //...
    if (!isViewVisible) {
        //...
    } else if (cancelAndRedraw) {
        //...
        // Try again
        scheduleTraversals();
    } else {
        //...
        //绘制
        if (!performDraw() && mActiveSurfaceSyncGroup != null) {
            mActiveSurfaceSyncGroup.markSyncReady();
        }
    }               
 }
```