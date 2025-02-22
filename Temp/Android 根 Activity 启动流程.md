# Android 根 Activity 启动流程
- 从 Launcher App 点击指定 APP 的图标去启动应用 


## Launcher

## Instrumentation

## ActivityManager 和 ActivityManagerService

## ActivityThread 和 ActivityThread$ApplicationThread





Launcher 点击 App 通过 AIDL Binder 进程间通信方式通知到 AMS（SystemServer 进程中）
    Launcher#startActivitySafely
    -Activity#startActivity 带上 FLAG_ACTIVITY_NEW_TASK 标记
    --Activity#startActivityForResult
    ---Instrumentation#execStartActivity
    ----ActivityManager.getService().startActivity
    ActivityTaskManager.getService().startActivity

AMS 想调用 ActivityThread$ApplicationThread 的方法
4 如果 ActivityThread$ApplicationThread 所在进程未启动，AMS 通过 Server Socket 方式通知 Zygote 进程去完成这个应用程序进程的创建
5 如果已经创建了，那么 AMS 通过 Binder 方式和 ActivityThread$ApplicationThread 所在进程进行通信，请求启动根 Activity




ActivityThread#performLaunchActivity 内部创建 Activity 的 Context，利用类加载器通过 Instrumentation#newActivity 创建对应 Activity 的实例，同时也创建了 Application
-Activity#attach 内部创建了 PhoneWindow 
-Instrumentation#callActiviyOnCreate
--Activity#preformCreate
---Activity#onCreate

ActivityThread#handleResumeActivity
```