# Android ANR 机制
- Application Not Responding 应用程序无响应，如果 Android 应用的 UI 主线程处于阻塞状态的时间过长（进行耗时的 I/O 的操作或者长时间的计算），如果应用位于前台，系统会向用户显示一个 ANR 对话框，提示用户应用无响应，为用户提供强行退出应用的按钮选项，相当于当系统检测到应用应用的 UI 主线程在一定时间内无法及时响应处理用户输入事件或绘制操作，就会触发 ANR

## 触发 ANR 的场景
- Activity：在 5 秒内未完成 onResume 或者在 5 秒内未响应输入事件（如按键或屏幕触摸）
- BroadcastReceiver： 前台广播的 onReceive 方法耗时超过大概 10 秒，后台广播超过大概 60 秒
- Service：前台服务的 onCreate、onStartCommand、onBind 等方法 20 秒内没有执行完成，后台服务则在 200 秒内
- ContentProvider：query、insert、publish 等方法在 10 秒内没有执行完成

## BROADCAST_TIMEOUT_MSG
- 发送广播时根据广播发送中的 intent 是否带有 FLAG_RECEIVER_FOREGROUND 标记可以分为前台广播和后台广播

```java
前台广播超时时间
- Android 13 及以下版本 10 秒
- Android 14 及以上版本，10-20 秒，具体取决于进程是否已耗尽 CPU
后台广播超时时间
- Android 13 及以下版本 60 秒
- Android 14 及以上版本，60-120 秒，具体取决于进程是否已耗尽 CPU
```

 














Input dispatching timed out 输入调度超时：应用在 5 秒内未响应输入事件（如按键或屏幕触摸）
Executing service
Service.startForeground() not called
Broadcast of intent
JobScheduler interactions

- 主线程在对另一个进程进行同步 binder 调用，而后者需要很长时间才能返回。
- 主线程处于阻塞状态，为发生在另一个线程上的长操作等待同步的块。
- 主线程在进程中或通过 binder 调用与另一个线程之间发生死锁。主线程不只是在等待长操作执行完毕，而且处于死锁状态。


StrictMode 严格模式

#### 启用后台 ANR 对话框

**开发者选项**中启用了**显示所有 ANR** 



#### TraceView

Android Device Monitor



```
/data/anr/traces.txt
//新版
/data/anr/anr_*
```



如果主线程无法继续执行，则它处于 `BLOCKED` 状态，并且无法响应事件。该状态在 Android Device Monitor 中会显示为“Monitor”或“Wait”



### 死锁

线程进入等待状态时会发生死锁，因为所需资源由另一个线程持有，而该线程也在等待第一个线程持有的资源。如果应用的主线程处于这种情况，很可能会发生 ANR




 BroadcastReceiver 中 onReceive 代码要尽量减少耗时，耗时建议使用 IntentService 处理



[ANR监测机制 - 简书 (jianshu.com)](https://www.jianshu.com/p/ad1a84b6ec69)
https://developer.android.google.cn/topic/performance/vitals/anr?hl=zh_cn
https://developer.android.google.cn/topic/performance/anrs/keep-your-app-responsive?hl=zh-cn
