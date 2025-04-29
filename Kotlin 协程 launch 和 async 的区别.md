# Kotlin 协程 launch 和 async 的区别
- launch 和 async 都是用于创建启动协程的函数，不阻塞当前线程，都是 CoroutineScope 接口的扩展函数
- 默认情况下 launch 和 async 的任务都是并行执行的
- launch 用于执行无需返回值的异步任务（比如日志上传、埋点和 IO 操作）
- async 用于执行并发任务并汇总结果的场景（比如多个接口并行请求后合并数据）

## launch
- launch 返回协程的 Job 对象，用于控制协程生命周期（比如取消协程、等待协程完成）
- Job 有一个 join 挂起函数，可以用于等待子协程运行完毕，有一个 cancel 函数可以取消协程
- 异常处理​：异常立即传播，导致协程取消（取消整个协程作用域），可通过 try-catch 捕获
```kotlin
val job = launch { 
    //执行后台任务
}
job.join() //等待协程完成，但不获取结果
```

```kotlin
suspend fun suspendFun_fetchDoc1(): String {
    delay(5000)
    return "text01"
}

suspend fun suspendFun_fetchDoc2(): String {
    delay(3000)
    return "text02"
}
//并行
suspend fun fetchTwoDocs() =
    coroutineScope {
        val job1 = launch(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val job2 = launch(Dispatchers.IO) { suspendFun_fetchDoc2() }
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是并行的
        job1.join()
        job2.join()
        println("fetchTwoDocs end")
    }
//串行
suspend fun fetchTwoDocs2() =
    coroutineScope {
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是串行的
        val job1 = launch(Dispatchers.IO) { suspendFun_fetchDoc1() }
        job1.join() //launch 后立即调用 join，相当于等待任务 1 完成后再执行任务 2
        val job2 = launch(Dispatchers.IO) { suspendFun_fetchDoc2() }
        job2.join()
        println("fetchTwoDocs2 end")
    }
```

## async
- async 返回协程的 Deferred 对象（Deferred 继承自 Job，额外提供 await 挂起函数用于获取结果，可以看成是一个带返回值的 Job）
- 通过 await 挂起函数获取返回结果，就是 async return 返回的值，如果不使用 await，那么效果就相当于 launch
- 异常处理​：异常暂存在 Deferred 中（持有异常并将其作为 await 调用的一部分重新抛出），直到调用 await 函数时抛出，这意味着如果在普通函数中使用 async 的话，就只能以静默方式丢弃异常，而这些丢弃的异常不会出现在崩溃指标中，也不会出现在 logcat 中
```kotlin
val deferred = async {
    //...
    "数据结果" //必须有返回值
}
val result = deferred.await() //阻塞等待结果
```

```kotlin
suspend fun suspendFun_fetchDoc1(): String {
    delay(5000)
    return "text01"
}
suspend fun suspendFun_fetchDoc2(): String {
    delay(3000)
    return "text02"
}
//并行
suspend fun fetchTwoDocs() =
    coroutineScope {
        val deferred1 = async(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val deferred2 = async(Dispatchers.IO) { suspendFun_fetchDoc2() }
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是并行的
        val result1 = deferred1.await()
        val result2 = deferred2.await()
        println("fetchTwoDocs result1=$result1 result2=$result2")
        //----------------- 打印 -----------------
        //fetchTwoDocs result1=text01 result2=text02
    }
//并行
suspend fun fetchTwoDocs2() =
    coroutineScope {
        val deferred1 = async(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val deferred2 = async(Dispatchers.IO) { suspendFun_fetchDoc2() }
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是并行的
        val resultList = awaitAll(deferred1, deferred2) //和 Collection#awaitAll 逻辑基本一致
        resultList.forEachIndexed { index, result ->
            println("fetchTwoDocs2 index=$index result=$result")
        }
        //----------------- 打印 -----------------
        //fetchTwoDocs2 index=0 result=text01
        //fetchTwoDocs2 index=1 result=text02
    }
//串行
suspend fun fetchTwoDocs3() =
    coroutineScope {
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是串行的
        val deferred1 = async(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val result1 = deferred1.await() //async 后立即调用 await，相当于等待任务 1 完成后再执行任务 2
        val deferred2 = async(Dispatchers.IO) { suspendFun_fetchDoc2() }
        val result2 = deferred2.await()
        println("fetchTwoDocs3 result1=$result1 result2=$result2")
        //----------------- 打印 -----------------
        //fetchTwoDocs3 result1=text01 result2=text02
    }
//串行
suspend fun fetchTwoDocs4() =
    coroutineScope {
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是串行的
        val result1 = withContext(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val result2 = withContext(Dispatchers.IO) { suspendFun_fetchDoc2() }
        println("fetchTwoDocs4 result1=$result1 result2=$result2" )
        //----------------- 打印 -----------------
        //fetchTwoDocs4 result1=text01 result2=text02
    }
```