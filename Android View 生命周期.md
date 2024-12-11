 # Android View 生命周期

 ## 回顾 Activity 生命周期
- 6个
```java
 onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```

## View 生命周期

```java
//默认 View 属性 visibility 是 VISIBLE 的情况
onFinishInflate —— onAttachedToWindow —— onWindowVisibilityChanged(VISIBLE) —— onMeasure —— onSizeChanged —— onLayout —— onDraw —— onWindowFocusChanged(true)  —— onWindowVisibilityChanged(GONE) —— onWindowFocusChanged(false) —— onDetachedFromWindow
```
- onFinishInflate 在 Activity#onCreate 方法调用后执行，当 View 及其子 View 从 xml 文件中加载完成后调用
- onAttachedToWindow 在 Activity#onResume 方法调用后执行，当 View 被添加到 Window 时候被调用，Window 就是指 PhoneWindow
- onWindowVisibilityChanged(VISIBLE) 紧跟着 onAttachedToWindow，不过需要注意的是锁屏的时候 onWindowVisibilityChanged(GONE) 不执行（PS：锁屏时候 Activity 只走 onPause 不会走 onStop）
- onMeasure、onSizeChanged、onLayout 和 onDraw 通常在 onWindowVisibilityChanged(VISIBLE) 方法调用后执行，所以在 Activity 的 onResume 方法里是无法获取 View 正确的宽高的
- onSizeChanged 是在 View 的大小发生改变后才会执行（屏幕旋转导致 View 所在的布局空间尺寸变化等场景），这里可以根据新的宽高重新调整绘制的一些布局、重新初始化与尺寸相关的资源等操作
- onWindowFocusChanged 官方推荐采用 onWindowFocusChanged(true) 回调来确定当前 View 所在的 Activity 是对用户可见的并且可交互的，所以这里可以获取到 View 正确的宽高
- 默认 View 属性 visibility 是 GONE 的情况下整个流程就不继续走 onMeasure、onSizeChanged、onLayout 和 onDraw 了
- 默认 View 属性 visibility 是 INVISIBLE 的情况下整个流程就不继续走 onDraw 了
- onDetachedFromWindow 是当 View 从 Window 中移除时（比如 Activity 被销毁的时候）会调用此方法，可以释放资源或取消监听器等操作
- 还有一个额外的 onVisibilityChanged，感觉不常用，暂时没有考虑进去


## 生命周期使用场景

### onAttachedToWindow
- 可以进行一些资源准备、注册监听器或开启一些动画相关等操作、
 
### onDetachedFromWindow
- 销毁自定义 View 使用的资源、取消之前注册的监听器和停止相关动画等操作，以避免可能发生的内存泄漏等问题

### onWindowFocusChanged
- 可以考虑在方法 hasWindowFocus 值是 true 的时候去获取当前 View 的宽高尺寸
- 可以考虑在 onWindowFocusChanged 回调中进行暂停或恢复动画的操作



 
 