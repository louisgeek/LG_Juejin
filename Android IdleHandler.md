# Android IdleHandler
- android.os.MessageQueue.IdleHandler 是一个接口，通常用于在消息队列空闲的时候执行低优先级的任务
- 可以实现延迟初始化一些不是马上需要用到的资源，可以运行一些低优先级任务，比如一些数据的预加载
- 不推荐执行较耗时的操作，比如会占用主线程的时间，从而导致界面卡顿和响应延迟
- 可以替换一些 Handler#postDelayed 使用的场景，因为通常延迟的时间比较不靠谱，改用 IdleHandler 反而更合理
- 通常主线程中 IdleHandler 用的比较多，不过子线程也是可以用的（自测可行但是遇到一些问题）

```java
    /**
     * Callback interface for discovering when a thread is going to block
     * waiting for more messages.
     */
    public static interface IdleHandler {
        /**
         * Called when the message queue has run out of messages and will now
         * wait for more.  Return true to keep your idle handler active, false
         * to have it removed.  This may be called if there are still messages
         * pending in the queue, but they are all scheduled to be dispatched
         * after the current time.
         */
        boolean queueIdle();
    }
```



```java
        MessageQueue.IdleHandler idleHandler = new MessageQueue.IdleHandler() {
            @Override
            public boolean queueIdle() {
                //消息队列空闲时处理逻辑
                
                //如果返回 keep = true 表示保持这个 IdleHandler 一直处于活跃状态，只要消息队列空闲了 queueIdle 方法就会被回调
                //如果返回 keep = false 那么这个 IdleHandler 就会被 remove 移除列表，也就是说当下次队列空闲的时候，不会继续被回调了
                return false;
            }
        };
        Looper.myQueue().addIdleHandler(idleHandler);


        //可以通过 removeIdleHandler 移除 IdleHandler，可以在合适的地方调用，以免出现内存泄漏
        Looper.myQueue().removeIdleHandler(idleHandler);
```
 

## 特点
- IdleHandler 的执行时机是不可控的，如果 MessageQueue 一直有待处理的消息，那么 IdleHander 的执行时机会很靠后
- IdleHandler 的目的是在消息队列空闲时执行一些轻量级的、不紧急的任务，不推荐进行较耗时的操作
- Android 系统的 GC 回收场景就使用了这个机制，当空闲的时候会去执行 GC 操作