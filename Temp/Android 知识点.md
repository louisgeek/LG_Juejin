





协程构建器


viewLifecycleOwner.lifecycleScope 绑定fragment 的onCreateView()到 onDestroyView()这个范围的生命周期


lifecycleScope 绑定 fragment 的整个生命周期onCreate()到onDestroy()这个范围的生命周期，生命周期范围会更长



//get方法通过operator可以简写为[Key]

 //plus方法通过operator可以简写为 + 



 CPS 变换使同步代码异步化，增加额外的 Continuation 类型的参数，用于函数结果值的返回



 https://www.cnblogs.com/jingdongkeji/p/17784848.html#:~:text=%E7%BB%B4%E5%9F%BA%E7%99%BE%E7%A7%91%EF%BC%9A%E5%8D%8F%E7%A8%8B%EF%BC%8C%E8%8B%B1%E6%96%87Coroutine%20%5Bk%C9%99ru%E2%80%99tin%5D%20%EF%BC%88%E5%8F%AF%E5%85%A5%E5%8E%85%EF%BC%89%EF%BC%8C%E6%98%AF%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A8%8B%E5%BA%8F%E7%9A%84%E4%B8%80%E7%B1%BB%E7%BB%84%E4%BB%B6%EF%BC%8C%E6%8E%A8%E5%B9%BF%E4%BA%86%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%AD%90%E7%A8%8B%E5%BA%8F%EF%BC%8C%E5%85%81%E8%AE%B8%E6%89%A7%E8%A1%8C%E8%A2%AB%E6%8C%82%E8%B5%B7%E4%B8%8E%E8%A2%AB%E6%81%A2%E5%A4%8D%E3%80%82,%E4%BD%9C%E4%B8%BAGoogle%E9%92%A6%E5%AE%9A%E7%9A%84Android%E5%BC%80%E5%8F%91%E9%A6%96%E9%80%89%E8%AF%AD%E8%A8%80Kotlin%EF%BC%8C%E5%8D%8F%E7%A8%8B%E5%B9%B6%E4%B8%8D%E6%98%AF%20Kotlin%20%E6%8F%90%E5%87%BA%E6%9D%A5%E7%9A%84%E6%96%B0%E6%A6%82%E5%BF%B5%EF%BC%8C%E7%9B%AE%E5%89%8D%E6%9C%89%E5%8D%8F%E7%A8%8B%E6%A6%82%E5%BF%B5%E7%9A%84%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80%E6%9C%89Lua%E8%AF%AD%E8%A8%80%E3%80%81Python%E8%AF%AD%E8%A8%80%E3%80%81Go%E8%AF%AD%E8%A8%80%E3%80%81C%E8%AF%AD%E8%A8%80%E7%AD%89%EF%BC%8C%E5%AE%83%E5%8F%AA%E6%98%AF%E4%B8%80%E7%A7%8D%E7%BC%96%E7%A8%8B%E6%80%9D%E6%83%B3%EF%BC%8C%E4%B8%8D%E5%B1%80%E9%99%90%E4%BA%8E%E7%89%B9%E5%AE%9A%E7%9A%84%E8%AF%AD%E8%A8%80%E3%80%82


# 结构化并发
 协程结构背后的机制称为结构化并发。与全局范围相比，它具有以下优势：

作用域通常负责子协程，其生命周期与作用域的生命周期相关联。
如果出现问题或用户改变主意并决定撤销操作，该范围可以自动取消子协程。
该范围会自动等待所有子协程完成。因此，如果 scope 对应于一个协程，则父协程在其 scope 中启动的所有协程都已完成之前不会完成。





有2种方式可以等待一个协程执行完成：
调用launch时会返回一个job实例，调用job的join方法
调用async时会返回一个Deferred(Job的一种类型)，调用Deferred的await方法

2.为什么在该代码中要写入join呢，如果不写这句话 不会输出GlobalScope里的内容 因为GlobalScope有自己的作用域


可以使用 join() 等待启动的协程完成，但它不会传播其异常。但是，崩溃的子协程也会取消其父协程，并出现相应的异常。


cancelAndJoin方法会取消协程并等待它完成。

invokeOnCompletion 
invokeOnCancellation

suspend fun test4() = suspendCancellableCoroutine<String> { cont ->
    val call = OkHttpClient().newCall(...)
    // 定义一个取消回调事件
    cont.invokeOnCancellation {
        // 取消请求
        call.cancel()
    }
}

Thread.join方法用于等待另一个线程完成。比如在主线程中调用t.join（t是另一个线程），主线程会阻塞，直到线程t执行完毕。而yield方法主要是关于当前线程主动让出 CPU 资源，与等待其他线程完成没有关系。

而join主要就是干了一件事，即等待设置了isActive=false的协程执行完成

java 的 yield


结构化并发

需要注意的是，yield 并不能保证一定会立即让出 CPU 资源。它只是向调度器发出了一个信号，表示当前线程愿意暂时让出 CPU 资源。调度器可能会忽略这个信号，让当前线程继续执行。
当有多个相同优先级的线程在竞争 CPU 资源，并且希望每个线程都能有比较公平的执行机会时，可以使用yield。例如，在一个简单的多线程模拟程序中，多个线程执行相似的任务，为了避免某个线程长时间占用 CPU 而导致其他线程饥饿（得不到执行机会），可以适时地调用yield方法。不过，在实际的复杂应用中，由于线程调度的不确定性，yield的使用效果可能很难准确预估，所以需要谨慎使用


- Nothing 类似 Java 的 Void 
- Nothing 可被访问到，Nothing? 一直是 null




Semaphore 
Phaser 



Fragment 作为 LifecycleOwner 的问题
无法保证 Activity/Fragment 停止后不继续执行启动，通过 Lifecycle 得到的状态来进行先判断再执行逻辑，能够规避该问题



所以，只要没有入侵更改版本号的，基本上都还是粘性事件，只是通过某种手段进行过滤。
- LiveData 即使没有活跃的观察者的情况下也会保持数据，当有新的观察者注册或者生命周期状态从非活跃变为活跃时，观察者会立即接收到最新的数据





转换LiveData：可以使用 Transformations 类对 LiveData 进行转换。
合并LiveData：可以使用 MediatorLiveData 来合并多个 LiveData 源。

通过 Transformations.map() 和 Transformations.switchMap() 方法，可以从一个 LiveData 转换为另一个 LiveData。

MediatorLiveData：这是一种特殊的 LiveData 类，它可以观察其他 LiveData 对象并做出反应。这允许你合并多个 LiveData 数据源。



1. 为什么Fragment中要使用viewLifecycleOwner代替this
在Android开发中，Fragment与Fragment中的View的生命周期并不一致。我们在使用一些可观察的数据（比如LiveData）时，需要让观察者（observer）准确感知Fragment中的View的生命周期，而不是Fragment本身的生命周期。基于这样的需求，Android专门构造了与Fragment中的View相对应的LifecycleOwner，也就是viewLifecycleOwner。以下是相关代码示例说明其重要性：
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
                        
原文链接：https://blog.csdn.net/weixin_35691921/article/details/136209456




