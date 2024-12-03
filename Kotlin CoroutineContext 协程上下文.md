# Kotlin CoroutineContext 协程上下文
- CoroutineContext 是一个接口，有个 CoroutineContext.Element 接口继承了 CoroutineContext 接口，定义了一系列协程执行的相关元素
- CoroutineContext 接口的设计和 Map 很相识，也是根据 Key 取值的，配合操作符重载（plus 加号、get 中括号）可以写出比较灵活的代码，如果子协程创建时指定了 CoroutineContext，则会合并，相同 Key 的值会被覆盖，总得来说集合包含了用户定义的一些各种不同 Element 元素对象，包括管理了协程的生命周期，线程调度，异常处理等
    - CoroutineName 代表协程的名称，通常可用于调试或者日志记录的时候的识别
    - Job 代表一个协程任务，算是已启动协程的句柄，可以用来跟踪协程的状态以及对它的控制，即用于管理协程的生命周期
    - CoroutineDispatcher（Dispatchers）用于协程处理线程调度，将任务分派到当的线程
    - CoroutineExceptionHandler 用于协程的异常处理，处理未捕获的异常
    - ContinuationInterceptor 的作用是拦截协程的，CoroutineDispatcher 也实现了 ContinuationInterceptor 接口
- EmptyCoroutineContext 是一个 object 类实现了 CoroutineContext 接口，一般作为默认的协程上下文来使用，比如 launch 或者 async 的第一个 CoroutineContext 参数默认就是 EmptyCoroutineContext
- CombinedContext 组合的上下文实现了 CoroutineContext 接口，由于 CoroutineContext 也重载了 plus 操作符，而 CombinedContext 就是 + 号组合得到的
- 也可以通过 minusKey 函数移除元素

## CoroutineName 协程名称
- CoroutineName 是一个 data 数据类继承了 AbstractCoroutineContextElement 类（实现了 CoroutineContext.Element 接口）
- 是用来给协程命名的，支持传入 name 参数，重写的 toString 方法可输出 "CoroutineName($name)" 格式字符串

## Job 任务
- Job 是一个接口，继承了 CoroutineContext.Element 接口，
- 拥有 start、 cancel 和 join 等方法，比如可以调用 job.cancel() 取消协程
- Deferred 接口继承了 Job

## CoroutineDispatcher（Dispatchers）协程调度器
- Dispatchers 是一个 object 类，调度器确定了协程在哪个线程或哪些线程上执行，它可以将协程限制在一个特定的线程中执行，或将它分派到一个线程池，也或者让它不受限地运行
- 其中抽象类 CoroutineDispatcher 继承了 AbstractCoroutineContextElement 类（实现了 CoroutineContext.Element 接口）和 ContinuationInterceptor 接口

Dispatchers 里的 CoroutineDispatcher 类型，可以看到它叫 Dispatchers 而不是 Threads，比如 Threads.Main，Threads.IO 等等
- 1 Dispatchers.Default 默认，对应 DefaultScheduler，为 JVM 共享线程池，$schedulerName-worker-${index}，默认前缀是 DEFAULT_SCHEDULER_NAME = "DefaultDispatcher"，比如 DefaultDispatcher-worker-1，比如用于图片裁剪、二维码计算和 Json 解析操作等
- 2 Dispatchers.Main  对应 MainCoroutineDispatcher，Android 主线程
- 3 Dispatchers.IO  对应 DefaultIoScheduler，IO 线程池，默认为 64 个线程，比如用于网络请求，文件读写，数据库操作等
- 4 Dispatchers.Unconfined  对应 kotlinx.coroutines.Unconfined，不限制，当前 CoroutineScope 的线程策略
- ExecutorService 的 asCoroutineDispatcher 扩展方法可以把线程池作为协程调度器，比如 Executors.newSingleThreadExecutor().asCoroutineDispatcher()
- Handler 的 asCoroutineDispatcher 扩展方法可以把 Handler 所在线程作为协程调度器，比如 Handler(Looper.getMainLooper()).asCoroutineDispatcher()

## CoroutineExceptionHandler 协程异常处理器
- CoroutineExceptionHandler 是一个接口，继承了 CoroutineContext.Element 接口
- 统一给父协程定义一个 CoroutineExceptionHandler，如果一个子协程异常了，其他子协程也不需要继续的话就采用 coroutineScope 方法，默认的 Job 就是这种表现，如果一个子协程异常了，其他子协程还需要继续的话就采用 supervisorScope 方法，或者创建协程作用域的时候传入带 SupervisorJob() 函数返回的特殊 Job 的协程上下文
- 子协程如果想单独处理异常就单独给子协程定义一个 CoroutineExceptionHandler

- 传入 exceptionHandler 方式和 supervisorScope
```kotlin
 val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        //CoroutineExceptionHandler#handleException 方法回调
        Log.d("exceptionHandler", "${coroutineContext[CoroutineName].toString()} 处理异常 ：$throwable")
    }
    GlobalScope.launch(exceptionHandler) {
        supervisorScope {
            launch(CoroutineName("异常子协程")) {
                Log.d("${Thread.currentThread().name}", "我要开始抛异常了")
                throw NullPointerException("空指针异常")
            }
            for (index in 0..10) {
                launch(CoroutineName("子协程$index")) {
                    Log.d("${Thread.currentThread().name}正常执行", "$index")
                    if (index %3 == 0){
                        throw NullPointerException("子协程${index}空指针异常")
                    }
                }
            }
        }
    }
```

- 创建 CoroutineScope(SupervisorJob() + exceptionHandler) 方式
```kotlin
val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        //CoroutineExceptionHandler#handleException 方法回调
        Log.d("exceptionHandler", "${coroutineContext[CoroutineName].toString()} 处理异常 ：$throwable")
    }
    //CoroutineScope 方法，SupervisorJob 方法代表子协程会自己处理异常，并不会影响其兄弟协程或者父协程
    val supervisorScope = CoroutineScope(SupervisorJob() + exceptionHandler)
    with(supervisorScope) {
        launch(CoroutineName("异常子协程")) {
            Log.d("${Thread.currentThread().name}", "我要开始抛异常了")
            throw NullPointerException("空指针异常")
        }
        for (index in 0..10) {
            launch(CoroutineName("子协程$index")) {
                Log.d("${Thread.currentThread().name}正常执行", "$index")
                if (index % 3 == 0) {
                    throw NullPointerException("子协程${index}空指针异常")
                }
            }
        }
    }
```

## ContinuationInterceptor 协程拦截器
- CoroutineDispatcher 也实现了 ContinuationInterceptor 接口，说明 CoroutineDispatcher 也具有拦截器的功能，所以默认情况下一旦使用了 Dispatchers.Main 等就会替换掉之前设置的自定义拦截器


