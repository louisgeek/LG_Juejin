# Android Activity 生命周期
> 官方文档 https://developer.android.google.cn/guide/components/activities/activity-lifecycle

## 核心回调
- 6 个核心的生命周期
```java
onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```
- onRestart 回调方法是在 Activity 从不可见（onStop）重新回到前台时调用的

![Activity 生命周期](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png "https://developer.android.google.cn/guide/components/images/activity_lifecycle.png")

- 所有生命周期回调方法重写时必须调用其父类方法
- 由于 Activity 经常在暂停和恢复之间来回切换，所以 onResume 和 onPause 的逻辑应该是轻量级的
- 图中显示系统在某些情况下回收内存导致 onStop 和 onDestory 可能不被调用，因此可以根据实际情况在 onPause 中保存一些非常重要的数据，耗时操作需要开子线程处理

## 经典场景
- 1 第一次启动 Activity A
```java
//Activity A
onCreate -> onStart -> onResume
```

- 2 在 Activity A 上启动 Activity B    
```java
//Activity A，如果 Activity B 是完全透明或是对话框主题的话，那么 A 就不继续走 onStop 了
onPause ->（A 被 B 覆盖不可见后走）onStop ->（如果 A 自身 finish 或被系统回收后走）onDestory
//Activity B
onCreate -> onStart ->（B 会等到 A 的 onPause 执行后走）onResume
```

- 3 从 Activity B 返回到 Activity A  
```java
//Activity B
onPause -> onStop -> onDestory
//Activity A
onRestart -> onStart -> onResume
//如果 Activity A 自身 finish 或被系统回收后，则这么走
onCreate（重新创建） -> onStart -> onResume
```

- 4 点击 HOME 键、来电
```java
//点击 Home 键、来电
onPause -> onStop 
//回到 App
onRestart -> onStart -> onResume  
//如果被系统回收了，则这么走
onCreate（重新创建） -> onStart -> onResume
```

- 5 锁屏与解锁
```java
//锁屏
onPause -> onStop
//解锁
onRestart -> onStart -> onResume
//如果被系统回收了，则这么走
onCreate（重新创建） -> onStart -> onResume
```

## 生命周期推荐做的事
### onCreate
- 应该尽量减少 onCreate 的工作量，避免程序启动太久而看不见界面，这里可以通过 savedInstanceState 参数恢复一些状态，如果 Activity 是第一次创建的话此时 savedInstanceState 为 null ，所以需要做判空处理
- 应该调用 setContentView 方法初始化布局
- 定义成员变量（全局变量），初始化相关数据
- 初始化视图控件等 UI 元素、配置 UI，将数据绑定到列表等 
- 可能需要将 Activity 与 ViewModel 相关联
- 可能需要启动与 Service 服务的绑定操作

### onStart
- 注册 BroadcastReceiver 广播接收器
- 地图导航位置更新等相关的初始化
- 把在 onStop 中释放的资源重新创建回来

### onResume
- 意味着此时 Activity 位于 Activity 堆栈的顶部，获取了焦点
- 把 onPause 中暂停的操作恢复回来，如 Camera 预览
- 开始动画、视频的播放
- 可能需要注册传感器（如 GPS）监听

### onPause
- 不推荐在这里保存应用或用户数据、进行网络请求调用或执行数据库事务（可根据实际情况在 onPause 中保存一些非常重要的数据），即不适合做耗时较长的工作，应最大程度减少 onPause 的工作量避免 Activity 切换缓慢卡顿
- 释放系统资源，传感器（如 GPS）句柄、Camera 预览等，通常是耗电的
- 停止动画、视频的播放
- 地图导航页面一般不在这里释放，因为希望它仍然能够继续工作

### onStop
- 可以在这里保存应用或用户数据（比如将用户编写的邮箱草稿保存到持久性存储空间）、进行网络调用、用户首选项持久性数据和执行数据库事务
- 应该释放那些不再需要的资源
- 关闭那些 CPU 执行相对密集的操作
- 地图导航可根据实际情况按需从精确位置更新切换到粗略位置更新
- 可以停止通过 Service 定时更新 UI 上的数据的 Service
- 取消注册 BroadcastReceiver 广播接收器

### onDestoty
- 应释放先前的回调（如 onStop）中尚未释放的所有资源
- 解除与 Service 服务的绑定
- 其实不推荐在 onDestroy 里执行销毁资源的工作，因为 onDestroy 执行的时机可能较晚，可根据实际需求在
onPause 或 onStop 中结合 isFinishing 判断来执行


## 特殊情况下的生命周期
- 1 系统资源不足的情况下导致 Activity 被系统回收
- 2 系统 Configuration 配置改变：横竖屏切换、系统语言改变和切换到多窗口或分屏模式等
- 3 用户强杀应用或者使用系统【设置】里的【应用管理器】来停止应用以终止进程
- 4 应用程序出现异常崩溃

### onSaveInstanceState
- 在 Activity 被系统销毁并重新创建后调用，通常会在 onStop 之前调用
- Instance State 实例状态，是一个键值对集合存储在 Bundle 对象中，保存着有关 Activity 的 View Hierarchy State 视图层次结构状态的瞬时信息（如输入框的值、列表滑动后停留的位置），系统用它来恢复 Activity 先前的状态，默认情况下系统使用 Bundle 对象中的实例状态来保存 Activity 布局中每个 View 对象的相关信息，系统因系统限制（例如 Configuration 配置变更或内存压力）等情况而销毁 Activity 的场景，当用户尝试回退到该 Activity 时，系统会通过已保存的实例状态去新建该 Activity 的实例，也就是整个过程 Activity 会先销毁再重建，即无需编写代码就能恢复布局状态为其先前的状态，主动调用 finish 方法和点击返回键的场景是不会调用该方法保存状态的
- super.onSaveInstanceState 里默认已经实现保存视图层次结构状态的逻辑
- Bundle 对象并不适合保留大量数据，在主线程中进行序列化和反序列化，会产生一定内存消耗，保存临时数据为主，保存简单轻量的界面状态，如果保存大量数据应该配合使用 ViewModel 进行处理

### onRestoreInstanceState
- 在 Activity 被系统销毁并重新创建后调用，用于恢复之前保存的状态数据，通常在 onStart 或 onResume 之后调用
- 可用于恢复一些 onSaveInstanceState 方法中保存的数据
- super.onRestoreInstanceState 已经实现恢复视图层次结构的状态的逻辑

### 横竖屏切换
- 如果是重建 Activity 的情况下，需要利用 onSaveInstanceState 和 onRestoreInstanceState 方法处理数据的保存与恢复，来保证用户数据或状态不丢失
- 如果是不重建 Activity 情况下，那么我们可以按需利用 onConfigurationChanged 回调进行判断来展示横竖屏不同界面的展示

```java
//Android 3.2 API 13 以前
不设置 Activity 的 android:configChanges 时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次
设置了 Activity 的 android:configChanges="orientation" 时，切屏还是会重新调用各个生命周期，切横、竖屏时都只会执行一次
设置了 Activity 的 android:configChanges="orientation|keyboardHidden" 时，切屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法

//Android 3.2 API 13 及以后
不设置 Activity 的 android:configChanges 时，或者设置了 Activity 的 android:configChanges="orientation" 时，或者设置了 Activity 的android:configChanges="orientation|keyboardHidden" 时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行一次
设置了 Activity 的 android:configChanges="orientation|screenSize|keyboardHidden"，切屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法
```

## 常见问题
1 如果在 onCreate 、onStart 和 onResume 等方法中直接调用 finish 方法，生命周期是怎样的？
- 系统会跳过对应的生命周期方法，可以简单理解 onCreate 和 onDestory ，onStart 和 onStop ，onResume 和 onPause 是对应的，原理可以参见源码里 Instrumentation 的判断逻辑

```java
//调用 finish 方法的时机
onCreate：onCreate -> onDestroy
onStart ：onCreate -> onStart -> onStop -> onDestroy
onResume：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy
```

2 什么时候只会走 onPause 方法，而不会走 onStop 方法？
- 打开一个完全透明或是对话框主题的 Activity 的情况
- 打开系统对话框（比如音量调节、亮度调节和通知权限弹窗等）
- 多窗口模式（分屏模式、画中画模式）

3 Activity 在什么时候可能会出现不执行 onDestory 方法的情况？
- 系统资源不足的情况下
- 用户强杀应用或者使用系统【设置】里的【应用管理器】来停止应用以终止进程
- 应用程序异常崩溃
- 切到多窗口或分屏模式

4 下拉状态栏时 Activity 的生命周期是什么？
- 不走任何生命周期，状态栏和 AlertDialog、Toast 等都是通过 WindowManager.addView 方法来显示的，对 Activity 的生命周期没有影响，另外可以通过 onWindowFocusChanged(boolean hasFocus) 来监听状态栏，hasFocus 为 false 可以表示下拉状态，从而可以实现暂停视频等需求

