# Android Handler 消息机制
- Handler 消息机制是设计用来进行线程间通信的一种机制，Android 系统是基于消息机制驱动运行的，这套消息处理机制是非堵塞的消息传递机制，是封装了一套消息创建、传递和处理等操作的机制
- 组件包括如下：
    - Handler 消息处理器
    - Looper 消息循环器（轮询器）
    - MessageQueue 消息队列，它是一个单向链表实现的队列，支持 FIFO 先进先出（一般情况下：链表是基于指针和节点，顺序表是基于数组实现的，而队列可以通过多种方式实现，比如数组和链表）
    - Message 消息
- 代码中 Handler 持有一个 Looper，Looper 持有一个 MessageQueue（PS：不过 Handler 也持有一个 Looper#MessageQueue 引用，应该是为了方便使用）

## Handler 的功能
- Handler 是负责发送和处理消息的工具组件
    - 1 在线程之间互发消息，实现线程间通信，通知主线程的 UI 更新，切换线程
    - 2 发送延时消息，延时任务，定时执行任务

## 消息流程
- 消息信息内容会被封装成 Message 消息对象，然后通过 Handler 发送添加到 MessageQueue 消息队列中，Looper 是一个消息循环器，用于不断地从消息队列中取出消息进行分发到 Message#target 里存着对应的 Handler 进行处理
    - 1 创建 Message 对象，可以直接 new 生成，或者推荐通过 Message#obtain 静态方法传入 handler 进行获取创建，复用现有 Message，避免频繁地创建 Message，消息池里最多能有 50 个 Message 作复用，如果消息池为空则内部会直接 new 出后返回 
    - 2 实例化 Handler 通过构造函数传入指定的 Looper 作关联，所以一个 Handler 对应一个 Looper，每个线程只能有一个 Looper，不过每个线程可以创建多个 Handler 实例
    - 3 发送消息一般用 Handler#sendMessage，可以通过 Handler#sendEmptyMessage 发送空消息，或者 post 发送 Runnable 任务，Runnable 会被封装成 Message，让任务在该 Handler 所在线程执行，所以发送消息基本都走 Handler#sendMessageAtTime 方法，然后最后调的都是 Handler#enqueueMessage 方法
    - 4 消息循环就是 Looper 的 loop 方法里有 for 循环不断通过 msg = queue.next 去取出消息，然后会调用 msg.target.dispatchMessage 将消息分发到关联的 Handler 终去处理
    - 5 消息分发会优先回调 Message#callback 执行 Runnable 的 run 方法，如果有就直接处理完 Runnable 后就结束了，所以 post 发送的都是走这个逻辑，没有就继续回调 Handler 构造函数中的 Callback，有就处理，而此时如果处理返回 true 那么流程就结束了，没有或者处理返回 false 则就继续回调 Handler#handleMessage ，所以可以重写 Handler#handleMessage 来处理接收到的消息

PS：post 和 send 通常最后都是调用 sendMessageAtTime 方法，然后是 enqueueMessage，因为存在多个线程同时往一个 MessageQueue 发送消息的可能，所以 enqueueMessage 内部肯定需要存在线程同步的逻辑（synchronized 关键字）

### 创建 Message
```java
//1 直接实例化
Message msg = new Message();
//2 传入 handler 后和下面写法一致，有缓存机制，避免重复创建 Message 实例对象
Message msg = Message.obtain(handler);
//3 内部直接调用 Message.obtain(this);
Message msg = handler.obtainMessage();
```

### 创建 Handler
```java
//1 Handler 无参构造函数已经被标记过时，内部默认是 mLooper = Looper.myLooper(); 这样不够明确，如果子线程调用，而此时 Looper 如果没有初始化，那么就会抛出异常
Handler handler = new Handler(); //@Deprecated
//2 显式传入 Looper，重写 Handler 的 handleMessage 方法
Handler handler = new Handler(Looper.myLooper());
//3 传入 Looper 和 Handler#Callback，重写 Handler.Callback 的 handleMessage 方法
Handler handler = new Handler(Looper.myLooper(), Handler#Callback);
//4 指定主线程
Handler uiHandler = new Handler(Looper.getMainLooper());
```
### 发送消息
1 Handler#sendMessage 方式

```java
handler.sendMessage(msg);
//sendMessage 就是调用了 sendMessageDelayed
handler.sendMessageDelayed(msg,delayMillis);
//
handler.sendEmptyMessage(what);
//sendEmptyMessage 就是调用了 sendEmptyMessageDelayed，而 sendEmptyMessageDelayed 调用了 sendMessageDelayed
handler.sendEmptyMessageDelayed(what,delayMillis);
```

2 Handler#post 方式
```java
//handler.post
public final boolean post(@NonNull Runnable r) {
    //sendMessageDelayed 和上文说的 sendMessage 里的原理一致
    return  sendMessageDelayed(getPostMessage(r), 0);
}
//handler.postDelayed
public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
//Runnable 被封装成 Message 的 callback
private static Message getPostMessage(Runnable r) {
   Message m = Message.obtain();
    //Message#callback 被赋值成 Runnable
   m.callback = r;
   return m;
}
```

3 Activity#runOnUiThread
```java
 public final void runOnUiThread(Runnable action) {
    if (Thread.currentThread() != mUiThread) {
     //内部也是调用 Handler#post
        mHandler.post(action);
    } else {
        action.run();
    }
}
```

### 消息循环
```java
//Looper#loop
public static void loop() {
    //...
    for (;;) {
     //内部会通过 msg = queue.next 去取出消息
    if (!loopOnce(me, ident, thresholdOverride)) {
        return;
     }
    }
    //...
}
private static boolean loopOnce(final Looper me, final long ident, final int thresholdOverride) {
    //从队列里取消息
    Message msg = me.mQueue.next(); // might block
    //...
    //target 就是消息关联的 Handler
    msg.target.dispatchMessage(msg);
    //...
}
```

### 消息分发
```java
/**
 * Handle system messages here.
 */
//Handler#dispatchMessage
public void dispatchMessage(@NonNull Message msg) {
    if (msg.callback != null) {
      //内部直接调用 message.callback.run()，所以就是用来处理 Handler#post 之类发送的消息
      handleCallback(msg); //对应处理【Handler#post 传入 Runnable】
    } else {
        //Handler#Callback 接口
        if (mCallback != null) {
            //对应处理【实现 Handler#Callback 接口 Callback#handleMessage 方法】
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //如果没有 mCallback 要处理，或者 mCallback 处理返回 false 那么才会继续走这里
        handleMessage(msg); //对应处理【重写 Handler#handleMessage】
    }
}
```


## Looper
- 在线程启动时，通常需要通过调用 Looper.prepare 创建 Looper，然后要调用 Looper.loop 启动消息循环，Looper.prepare 里 Looper 构造的时候会创建了一个 MessageQueue，同时 Looper.loop 方法里会启动 MessageQueue 队列的循环运转，Looper.prepare 里的逻辑也表明了此方法在同一个线程中是不能重复执行的，也就说明每个线程对应都只能有一个 Looper，所以也就决定了每个 Looper 都只有一个消息队列作关联，另外就是 new 出的 Looper 会存入 sThreadLocal，后续可以通过 Looper#myLooper 获取当前线程的 Looper，其实就是从 sThreadLocal 里拿的
- 主线程日常不需要调用 Looper.prepare 和 Looper.loop 操作，那是因为安卓系统内部的 ActivityThread#main 方法已为我们执行了 Looper#prepareMainLooper 和 Looper#loop 代码，也正是因为有这些逻辑，安卓系统才能运行起来，而专用的 Looper#prepareMainLooper 里的逻辑也表明了该方法不能再次调用，内部通过调用 Looper#prepare 方法建完后，还额外保存到了静态变量 sMainLooper 中，方便后续直接通过 Looper#getMainLooper 获取使用


PS：ThreadLocal$ThreadLocalMap 是用于存储线程相关的一些私有属性的 Map，这个 Map 的 key 是 ThreadLocal 对象，value 是要存储的对象，而 Handler 中就是通过 ThreadLocal 机制来保证一个线程只有一个 Looper 对象的


## 线程切换
- Handler 可以将消息或任务发送添加到与创建 Handler 时关联的线程的 Looper 的 MessageQueue 消息队列中去执行，这就意味着在消息机制各组件之间的协同工作下，实现了可以在不同线程间传递消息并在指定线程处理消息的功能，从而达到了线程切换的效果

```kotlin
val mainHandler = Handler(Looper.getMainLooper())
//子线程调用
mainHandler.post { 
   //主线程 UI 操作
}
```
 
## Asynchronous Message 异步消息和 SyncBarrier 同步屏障
- Message 大致可以分成普通消息（同步消息）、异步消息、和屏障消息（同步屏障）
- 异步消息在创建 Message 对象时，可以通过设置 Message#setAsynchronous(true) 来标记这条消息为异步消息
- 同步屏障的特点是 msg.target = null，就是没有关联 Handler 的消息（我们正常发送普通消息时 target 是不会出现为 null 的情况的），相当于一个标志位，通过 MessageQueue#postSyncBarrier 发送，通过 MessageQueue#removeSyncBarrier 进行移除，是系统内部使用的机制，上层开发一般不会直接使用到
- 两者都是特殊的 Message 对象，同步屏障的作用就是为了保证异步消息可以优先执行，当出队列操作遇到队头为同步屏障时，则其后面的同步消息将暂时不会被执行，只会优先执行异步消息，直至同步屏障被移除，常见应用就是 ViewRootImpl#scheduleTraversals 方法里会通过 mHandler.getLooper().getQueue().postSyncBarrier() 添加同步屏障以确保 UI 绘制任务能够被及时执行，不造成界面卡顿


## HandlerThread
就是一个自带 Looper 的线程，说白了就是子线程在使用 Handler 需要手动处理 Looper.prepare 和 Looper.loop 等操作，而 HandlerThread 就是为了方便我们处理使用，通过使用 HandlerThread 直接创建带有消息队列的 Thread，常用于单线程串行执行多个耗时任务的场景，避免频繁地创建线程，也保证了线程安全，不过记得需要在合适的调用 quit 或 quitSafe 方法来结束线程以避免可能出现的内存泄露

## IdleHandler 空闲处理器
IdleHandler 是一个 MessageQueue 里的接口，通常用于在消息队列空闲的时候去执行一些低优先级、轻量级的任务，可以实现一些延迟初始化的应用