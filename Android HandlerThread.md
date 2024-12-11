# Android HandlerThread
- HandlerThread 继承自 Thread，组合了一个 Looper 和 Handler（目前是隐藏的），本质上是一种可以直接使用 Handler 机制（主要是 Looper）的线程
- HandlerThread 就是给开发者提供了一个比较简单方便的方式，去让我们在执行异步任务或长时间运行的耗时操作时，不阻塞 UI 主线程
- HandlerThread 简化了 Looper 的创建和管理，支持在一个单独的线程中处理消息队列，而自带的消息队列同时也确保了消息处理的顺序性，同时每个线程关联的独立的 Handler 进行顺序执行任务的特性以保证了线程安全，避免了在多个线程中同时操作同一个 Handler 的实例而引发的线程安全问题


## 实例化 HandlerThread
- 相当于创建了一个线程（子线程）
```java
HandlerThread handlerThread = new HandlerThread("name");
```

## 启动线程
```java
handlerThread.start() 
```

## 创建 Handler
- 传入 handlerThread 的 Looper 进行关联
```java
//Handler 无参方法已过时
Handler workHandler = new Handler(handlerThread.getLooper());
//可选覆写 handleMessage 方法执行具体的耗时任务
Handler workHandler = new Handler(handlerThread.getLooper()){
       @Override
       public void handleMessage(@NonNull Message msg) {
           super.handleMessage(msg);
       }
};
//带 callback
Handler workHandler = new Handler(handlerThread.getLooper(), new Handler.Callback() {
        @Override
        public boolean handleMessage(@NonNull Message msg) {
        //如果返回 true，那么就不会调用 Handler 内部的 handleMessage 方法处理
        return false;
     }
});
  ```

## 发消息
- 通过正常 Handler 的方式发消息
```java
Message msg = workHandler.obtainMessage();
workHandler.sendMessage(msg);
//
workHandler.post(new Runnable() {
   @Override
   public void run() {
                // ...
   }
})
```

  

## 退出
- 不再需要 HandlerThread 时记得在合适的地方停止它，比如 Activity 的 onDestroy 里
```java
//quit 是立即尝试退出，可能留下未处理的消息
HandlerThread.quit();
//quitSafely 会等待所有已经排队的消息被处理完之后再退出
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
  handlerThread.quitSafely();
}
```

  

