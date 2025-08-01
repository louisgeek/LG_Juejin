# Android ScaleGestureDetector 和 GestureDetector 手势检测
- ScaleGestureDetector.SimpleOnScaleGestureListener 静态内部类，实现了 OnScaleGestureListener 接口，方法都是默认空实现
- GestureDetector.SimpleOnGestureListener 静态内部类，实现了 OnGestureListener, OnDoubleTapListener, OnContextClickListener，方法都是默认空实现

## ScaleGestureDetector 缩放手势检测
- 可以接管 View 的 onTouchEvent 方法或者 OnTouchListener 的 onTouch 方法
```kotlin
   override fun onTouchEvent(event: MotionEvent?): Boolean {
        //return super.onTouchvent(event)
        return scaleGestureDetector.onTouchEvent(event) || super.onTouchEvent(event)
   }
```

## ScaleGestureDetector.OnScaleGestureListener
- OnScaleGestureListener 接口包括 onScale、onScaleBegin 和 onScaleEnd 方法
```java
public interface OnScaleGestureListener {
    //两个手指在屏幕上移动进行缩放手势的时候会持续调用，可以通过 detector.getScaleFactor() 获取缩放因子（比如 1.05 表示放大 5%，0.95 表示缩小 5%），来进行视图缩放逻辑的处理
    public boolean onScale(@NonNull ScaleGestureDetector detector);
    //缩放手势开始    
    public boolean onScaleBegin(@NonNull ScaleGestureDetector detector);
    //缩放手势结束
    public void onScaleEnd(@NonNull ScaleGestureDetector detector);
}
```

## GestureDetector 手势检测
- 可以接管 View 的 onTouchEvent 方法或者 OnTouchListener 的 onTouch 方法
```kotlin
   override fun onTouchEvent(event: MotionEvent?): Boolean {
        //return super.onTouchEvent(event)
        return gestureDetector.onTouchEvent(event) || super.onTouchEvent(event)
   }
- GestureDetector.OnContextClickListener 接口，外接键盘、蓝牙外设等事件
```

## GestureDetector.OnGestureListener
- OnGestureListener 接口包括 onScroll 滑动、onFling 抛、onSingleTapUp 短按和 onLongPress 长按等
```java
public interface OnGestureListener {
    //手指按下
    boolean onDown(@NonNull MotionEvent e);
    //手指按下后短时间内未移动或未抬起即触发
    void onShowPress(@NonNull MotionEvent e);
    //手指按下后抬起时触发（单击事件，返回 true 表示消费了这个事件）
    boolean onSingleTapUp(@NonNull MotionEvent e);
    //手指滑动时持续调用（返回 true 表示消费了这个事件），distanceX 和 distanceY 表示当前事件和上一个事件之间的移动距离（不是总距离）
    //distanceX 横向滑动距离，向右滑动为负数，x 值减小，向左滑动正数，x 值增加（prevX - currX），distanceX 纵向滑动距离，向下滑动负数，y 值减小，向上滑动正数，y 增加(prevY - currY)
    boolean onScroll(@Nullable MotionEvent e1, @NonNull MotionEvent e2, float distanceX, float distanceY);
    //手指长按（未滑动）
    void onLongPress(@NonNull MotionEvent e);
    //快速滑动后抬起手指（惯性滑动、抛、甩，返回 true 表示消费了这个事件）
    boolean onFling(@Nullable MotionEvent e1, @NonNull MotionEvent e2, float velocityX, float velocityY);
}
```

## GestureDetector.OnDoubleTapListener
- OnDoubleTapListener 接口包括 onSingleTapConfirmed 单击和 onDoubleTap 双击等
```java
public interface OnDoubleTapListener {
    //确认是单击（而不是双击其中的一次点击，有双击事件监听情况下紧跟着 OnGestureListener#onSingleTapUp 后面根据条件触发）
    boolean onSingleTapConfirmed(@NonNull MotionEvent e);
    //手指双击时候触发（双击事件，ACTION_DOWN 事件内回调）     
    boolean onDoubleTap(@NonNull MotionEvent e);
    //手指双击过程中的事件（ACTION_DOWN、ACTION_MOVE 和 ACTION_UP 事件内回调）         
    boolean onDoubleTapEvent(@NonNull MotionEvent e);
}
```

## GestureDetector 和 ScaleGestureDetector 组合使用
- 双指缩放：主要通过 ScaleGestureDetector#onScale 配合 Matrix#postScale 实现
- 拖动滑动：主要通过 GestureDetector#onScroll 配合 Matrix#postTranslate 实现
```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    scaleGestureDetector.onTouchEvent(event) //进行缩放处理
    gestureDetector.onTouchEvent(event) //比如进行滑动处理
    return true //简单粗暴
}
```
