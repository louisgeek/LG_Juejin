
# Android Binder IPC 进程间通信机制
- Binder 粘合剂：顾名思义就是粘和不同的进程，使之实现通信
- 进程隔离：进程与进程间内存是不共享的，一个进程无法直接另一个进程的数据，所以要引入进程间通信
- IPC：Inter-Process Communication 进程间通信，是指在不同进程之间进行数据交换、共享资源和信息传递的机制
- Binder 是 Android 提供的一种 IPC 进程间通信机制，是一种高效、安全、易用的跨进程通信方式
- Binder 是一种基于消息传递的 IPC 机制，其核心就是一个内核驱动程序，它负责在不同进程之间传递消息，使用 C/S 架构（服务端/客户端架构），易用性高
- 常用的系统服务 ActivityManagerService、PackageManagerService 等服务都是通过 Binder 方式和应用程序之间进行通信的
- 音乐播放器应用场景：可以把播放服务放在一个独立的进程中，通过 Binder 实现主进程与播放服务进程之间的通信，主进程可以发送播放、暂停、切换歌曲等指令给播放服务进程，播放服务进程可以将当前播放的状态回传给主进程进行显示，充分利用多核处理器的优势，总体来说多进程能带来更好的性能和用户体验


## 常见的 IPC 机制
- Shared Memory 共享内存
- Pipe 管道
- Message Queue 消息队列
- Socket 套接字：在同一主机上的不同进程之间通信，也可以支持在不同主机上的进程之间进行通信


## Binder IPC 通信机制
- 进程空间划分为 User Space 用户空间 和 Kernel Space 内核空间两种，简单来说内核空间就是系统内核运行的空间，用户空间就是用户程序运行的空间
- Client、Server 和 ServiceManager 三者之间交互都是基于 Binder 驱动进行通信的，三者进程都属于进程空间的用户空间，不可进行进程间交互，而 Binder 驱动属于进程空间的内核空间，可进行进程间交互，三者都是通过 Binder 驱动在内核空间进行拷贝数据
- Server 进程先注册一些 Service 到 ServiceManager 中，某个 Client 进程要使用某个 Service，必须先到 ServiceManager 中获取该 Service 的相关信息，Client 根据得到的 Service 信息建立与 Service 所在的 Server 进程通信的通路，然后就可以与 Server 进行交互了
- Client 进程只不过是持有了 Server 端的 Proxy 代理，代理对象协助 Binder 驱动完成了跨进程通信
- ServiceManager 管理 Service 的注册和查询
- Binder 驱动，Binder IPC 机制中涉及到的内存映射，它是通过 mmap() 来实现的


## AIDL Android 接口定义语言
- 用来抽象化 Binder IPC 的工具（就是接口的定义，说白了就是借助编译器生成 Binder 相关代码去实现，可以少些一些模板代码），可以理解为进程之间通信传输数据的桥梁
- 需要确保服务端和客户端的 AIDL 接口文件完全一致

```groovy
//build.gradle
android {
    //...
    buildFeatures {
        aidl true
    }
    //...
}
```

服务端：
- 1 AS 创建 aidl 文件（创建完后需要 build 一下生成同名 java 文件）
- 2 自定义 Service 类，onBind 方法传入 IBinder 的实例（实现 Xxx.Stub 的接口方法）
- 3 在 AndroidManifest.xml 文件中注册 Service

```java
// \app\src\main\aidl\com\louisgeek\louisaidlserver\IMyAidlInterface.aidl

// IMyAidlInterface.aidl
package com.louisgeek.louisaidlserver;

// Declare any non-default types here with import statements

interface IMyAidlInterface {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```

```kotlin
class MyAidlService : Service() {
    companion object {
        private const val TAG = "MyAidlService"
    }
    //private val binder = MyAidlInterfaceBinder()
    //private class MyAidlInterfaceBinder : IMyAidlInterface.Stub() {
    //}
    private val binder: IMyAidlInterface.Stub = object : IMyAidlInterface.Stub() {
        override fun basicTypes(
            anInt: Int,
            aLong: Long,
            aBoolean: Boolean,
            aFloat: Float,
            aDouble: Double,
            aString: String?
        ): String {
            val result = "$anInt $aLong $aBoolean $aFloat $aDouble $aString"
            Log.e(TAG, "basicTypes: louis===")
            return "MyAidlService服务端：result=$result"
        }
    }

    override fun onBind(intent: Intent): IBinder {
        Log.e(TAG, "onBind: louis===")
        return binder
    }
}
```

```xml
<service
            android:name=".MyAidlService"
            android:exported="true">
            <intent-filter>
                <action android:name="actionMyAidlService" />
            </intent-filter>
        </service>
```


客户端：
- 1 AS 创建 aidl 文件（和服务端一样，也可以直接复制过来，然后 build 一下）
- 2 调用 bindService 绑定远程 Service，传入自定义 ServiceConnection 实例
- 3 在 onServiceConnected 方法中 Xxx.Stub.asInterface(service) 获取到 IBinder 对象

```kotlin
    private var iMyAidlInterface: IMyAidlInterface? = null
    private val serviceConnection = object : ServiceConnection {
        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service)
        }

        override fun onServiceDisconnected(arg0: ComponentName) {
            iMyAidlInterface = null
        }
    }

    private fun initView() {
        val tv: TextView = findViewById(R.id.tv)
        tv.setOnClickListener {
            val result = iMyAidlInterface?.basicTypes(1, 2, true, 3.0F, 4.0, "5.0")
            Toast.makeText(this, "result=$result", Toast.LENGTH_LONG).show()
        }
    }

    override fun onStart() {
        super.onStart()
        val intent = Intent("actionMyAidlService")//服务端action
        intent.setPackage("com.louisgeek.louisaidlserver")//服务端包名
        val result = this.bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE)
        Log.e("TAG", "onStart: louis==result=$result" )
    }

    override fun onStop() {
        super.onStop()
        this.unbindService(serviceConnection)
    }
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <!-- Android 11 及以上需要声明软件包可见性需求 -->
    <queries>
        <package android:name="com.louisgeek.louisaidlserver" />
    </queries>
    <!-- ... -->
</manifest>
```

## IBinder、IInterface、Binder、BinderProxy 和 Stub
- android.os.IBinder 是一个接口，代表了一种跨进程传输的能力，只要实现了这个接口，驱动底层就能够支持对这个对象进行跨进程传输
- android.os.IInterface 是一个接口，编译工具自动生成的 aidl 文件同名接口会继承自 IInterface 接口，代表的就是远程 Server 对象具有什么能力，而能力就是 aidl 文件里定义的接口方法
- android.os.Binder 是 Java 层的 Binder 类（实现了 IBinder 接口），代表的就是 Binder 本地对象
- android.os.BinderProxy 是 Binder 类的一个内部类（实现了 IBinder 接口），代表远程进程的 Binder 对象的本地代理
- Xxx.Stub 是编译工具自动生成的 aidl 文件同名接口里的静态内部类，它是一个继承了 Binder 的抽象类（同时也实现了编译工具自动生成的 aidl 文件同名接口），具有远程 Server 承诺给 Client 的能力，当然这个能力的具体内容需要我们自己实现
- Xxx.Stub.Proxy 是编译工具自动生成的 aidl 文件同名接口里的静态内部类，也实现了编译工具自动生成的 aidl 文件同名接口，同时组合了一个 IBinder 对象（是 IBinder 代理对象，也就 Binder 代理对象）,代表远程进程的 Binder 对象的本地代理，对应BinderProxy？

固定套路：
- 一个需要跨进程传输的对象一定实现了 IBinder这个接口，如果是 Binder 本地对象，那么一定继承了 Binder 然后实现 IInterface 子接口，如果是代理对象，那么就实现了 IInterface 子接口并持有了 IBinder 引用（代理）
- 服务端：Service#onBind 返回 IBinder 对象，Server 端实现了 Xxx.Stub（继承 Binder（实现 IBinder））抽象类
- 客户端：bind 一个 Service 之后，在 onServiceConnected 的回调里通过 asInterface 这个方法返回 Xxx.Stub.Proxy 就是拿到了一个远程的 Service


 ## Android 基于 Linux 内核，为啥不直接采用 Linux 中现有实现的 IPC 机制
- 性能：Binder 数据拷贝只需要一次，有开销低、速度快、耗电少等优势（管道、消息队列和 Socket 都需要两次，但共享内存方式一次内存拷贝都不需要，从性能角度看，Binder 性能仅次于共享内存），Binder 相对出传统的 Socket 方式显得更加高效
- 安全鉴权：传统的跨进程通信方式对于通信双方的身份并没有做出严格的验证，而 Binder 机制从协议本身就支持对通信双方做身份校检，也是 Android 权限模型的基础，它为每个进程都分配了自己的 UID/PID 作为标识，它是鉴别进程身份的重要标志，有效提高了安全性
- 简单易用：Binder 使用面向对象的方式设计，使用 C/S 架构，相互独立、职责明确、架构清晰和易于使用
- 其他：Binder 是安卓架构师基于自己的开源项目 OpenBinder 重新实现的 Android IPC 机制，优先采用现成的          

两次拷贝：指的是需要先将数据从发送方进程拷贝到内核缓存区（内核空间内），然后再将数据从内核缓存区拷贝到接收方进程
内存映射：指的是将用户空间的一块内存区域映射到内核空间，当映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间，反之内核空间对这段区域的修改也能直接反应到用户空间，内存映射能够减少数据拷贝的次数，实现用户空间和内核空间的高效互动，发送方进程将数据拷贝到内核缓存区（内核空间内），由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间内，这样便完成了一次进程间的通信，只进行了一次拷贝

 ## 为啥 Zygote 进程中的 IPC 采用的是 Socket 机制
- 初始化时机：首先 Socket 机制是基于跨平台网络协议的，不依赖于 Android 特有的 Binder 机制，因此可以在 Binder 驱动初始化之前进行通信，很难保证 Zygote 进程去注册 Binder 的时候 ServiceManager 已经初始化好了，如果去做额外的同步工作就加大了复杂度了
- Zygote 的 fork 行为：Linux 中 fork 进程是不推荐去 fork 一个多线程的进程的，而 Binder 通常是多线程的，所以该场景下不推荐引入 Binder 机制，而且如果使用 Binder 的话，从 Zygote 中 fork 出子进程时就也会拷贝一份 Zygote 中的 Binder 对象，会造成额外的内存占用
 