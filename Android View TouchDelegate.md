# Android View TouchDelegate
- TouchDelegate 用于扩大某个 View 的触摸区域，将父容器指定的矩形区域内的触摸事件转发给目标 View 处理，当用户点击这块指定区域时，事件会被转发到目标 View 的 onTouchEvent 方法中处理，当然也可以通过 TouchDelegate 缩小子 View 的触摸区域
- TouchDelegate 必须设置在目标视 View 的父容器上，而非 View 本身，一个父容器只能设置一个 TouchDelegate
- 扩展区域的坐标是相对于父容器的，需确保计算后的区域不超出父容器边界
- 其他方案：设置 Padding、​自定义 onTouchEvent 逻辑

```kotlin
fun View.expandTouchRect(expandSize: Int) {
    val parentView = parent as View
    parentView.post {
        val rect = Rect()
        //获取目标 View 在其父视图中的当前原始边界
        getHitRect(rect)
        //扩展区域边界
        rect.left -= expandSize
        rect.top -= expandSize
        rect.right += expandSize
        rect.bottom += expandSize
        //扩展区域边界
        //rect.inset(-expandSize, -expandSize)
        //确保扩展后的矩形不会超过父视图的边界
        val parentRect = Rect()
        parentView.getHitRect(parentRect)
        val result = rect.intersect(parentRect)
        //在父容器中设置委托（代理）区域
        parentView.touchDelegate = TouchDelegate(rect, this)
    }
}
fun View.resetTouchRect() {
    val parentView = parent as View
    //通过设置空的 Rect 来恢复视图的原始触摸区域
    parentView.touchDelegate = TouchDelegate(Rect(), this)
}
```