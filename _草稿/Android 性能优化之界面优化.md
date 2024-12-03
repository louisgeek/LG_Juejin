# Android 性能优化
 


## 界面优化

### 过度绘制
打开开发者选项 -> 调试 GPU 过度绘制

层级嵌套


### 界面卡顿检测

BlockCanary 界面卡顿监控工具
- 基于 Looper，内部使用了 Looper.getMainLooper().setMessageLogging，利用主线程 Looper 打印的日志


Choreographer 掉帧（帧率下降、卡顿延迟）检测

```kotlin
object ChoreographerHelper {
    private const val TAG = "ChoreographerHelper"
    private const val SKIPPED_FRAME_WARNING_LIMIT = 30
    
    fun start() {
        //系统每 16.67ms （60Hz 每秒刷新 60 次,1000/60）发出一个 VSYNC 信号来通知刷新一次屏幕
        //假设在界面不卡顿的情况下，界面应该至少间隔 16.67ms 刷新一次，因此理论上至少每 16.67ms 应该会触发一次回调
        //如果发生界面卡顿了 ，那么回调的触发时间间隔就会超过 16.67 ms
        Choreographer.getInstance().postFrameCallback(object : FrameCallback {
            private var lastFrameTimeNanos = 0L
            private val frameIntervalNanos = (1.0F * 1000000000 / 60).toLong() //60Hz

            override fun doFrame(frameTimeNanos: Long) {
                if (lastFrameTimeNanos == 0L) {
                    lastFrameTimeNanos = frameTimeNanos
                }
                val jitterNanos = frameTimeNanos - lastFrameTimeNanos
                if (jitterNanos >= frameIntervalNanos) {
                    //计算掉帧数，就是延迟了多少帧
                    val skippedFrames = jitterNanos / frameIntervalNanos
                    if (skippedFrames > SKIPPED_FRAME_WARNING_LIMIT) {
                        //掉帧30以上的，可以上报日志
                        Log.e(TAG, "doFrame: 掉帧30以上的 skippedFrames=$skippedFrames")
                    }
                }
                lastFrameTimeNanos = frameTimeNanos
                //注册下一帧回调
                Choreographer.getInstance().postFrameCallback(this)
            }
        })
    }
}
```

JankStats metrics 


 