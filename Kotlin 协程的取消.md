# Kotlin Coroutine 协程的取消
- 协程可以通过判断检查 isActive 活跃状态状态并主动抛出一个 CancellationException 异常，或者使用 ensureActive 、yield 函数配合 cancel 函数实现协程的取消
- 协程的取消机制是一种协作式机制，是需要代码层面做配合来实现的，线程有协作式中断机制，而协程也有协作式取消的概念
- 另外处于取消状态的协程不能再被挂起，协程本身必须等到其所有子协程完成（处于完成或者取消状态）后才能完成

## CoroutineScope#cancel
- 如果用 CoroutineScope#cancel 取消一个 CoroutineScope 协程作用域的话，那么该作用域中的所有未完成的协程都将会被取消（实际上就是取消了该作用域对应的 Root Job，因为每个 CoroutineScope 内部持有的 CoroutineContext 协程上下文都会包含了一个 Job 对象，所以 CoroutineScope#cancel 和 Job#cancel 的取消理念是相通的）
- 一旦 CoroutineScope 协程作用域被取消了，那么就不能在此作用域中去启动新的协程了，所以相当于做到了防止新的协程在这个作用域中启动
- CoroutineScope#cancel 取消协程作用域并不保证协程立即停止

```kotlin
//每个 CoroutineScope 都对应着一个 Job，这个 Job 就会成为该协程作用域内所有协程的父 Job（理解为 Root Job），如果父 Job 被取消，那么自身和其所有子 Job 也会被递归取消
public fun CoroutineScope.cancel(cause: CancellationException? = null) {
    //意味着这个 Job 决定了该协程作用域内所有协程的生命周期
    val job = coroutineContext[Job] ?: error("Scope cannot be cancelled because it does not have a job: $this")
    job.cancel(cause)
}
```

## Job#cancel
- Job#cancel 函数只能取消特定的协程自身和其子协程（取消一个协程时，会递归取消所有子协程），但不会影响父协程，也不会影响到其兄弟协程（同级协程）
- Job#cancel 取消协程也不保证协程立即停止

## 协程的协作式取消
- 协程通过调用 job.cancel 来开始取消操作（类似线程的 Thread.interrupt 操作），不过这并不意味着协程会立即停止，如果只是单纯调用 cancel 而没有实际感知 cancel 操作的代码逻辑的话（手动结束协程的代码），那么协程会继续执行完所有任务后正常结束
- 通常协程自身需要定期检查 isActive 状态（Job 是否处于活跃状态，因为当协程被取消时这个 isActive 状态将会被置为 false）并抛出 CancellationException 这个特殊的异常或者 ensureActive 逻辑去配合 job.cancel 来完成协程取消操作（类似线程的 Thread.isInterrupted 判断）
- 线程存在阻塞状态（比如 Thread#sleep）下可以响应中断，同理协程调用 cancel 也会引起 delay 等函数会从等待中立即恢复过来并抛出 CancellationException 异常来让系统来取消协程
- 如果被取消的协程有 finally 块（比如 try-finally 包住一个 delay 函数），那么这个 finally 块中的代码依然会执行，可以用于清理资源等操作

协程任务执行完后正常结束
```kotlin
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        //模拟协程执行任务
        for (i in 0 until 100000) {
            println("协程运行中 i=$i")
        }
        //在执行完所有任务后协程正常结束
        println("---- 协程结束 ----");
    }
}
```

加入 job.cancel 取消协程的逻辑
```kotlin
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        //模拟协程执行任务
        for (i in 0 until 100000) {
            println("协程运行中 i=$i")
        }
        //虽然调用了 job.cancel 函数取消协程但还是会执行完所有的任务
        println("---- 协程结束 ----");
    }
    //演示逻辑
    delay(10)
    //等待一段时间后去取消协程
    job.cancel()
}
```

说明仅仅调用 job.cancel 是达不到取消协程的效果的，需要继续加入实现停止协程的逻辑
```kotlin
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        //模拟协程执行任务
        for (i in 0 until 100000) {
            //目的就是让协程在适当的时候检查 Job 是否处于活跃状态，按需处理收尾清理工作并完成停止协程的逻辑
            if (!isActive) { //类似 Thread#isInterrupted 的作用
                //释放资源并结束协程
                println("检测到 Job 处于不活跃的状态，于是准备停止")
                throw CancellationException() //通过抛出一个特殊的异常实现协程的取消，系统会处理这个特殊的异常去正常取消协程
            }
            println("协程运行中 i=$i")
        }
        //
        println("---- 协程结束 ----");
    }
    //演示逻辑
    delay(10)
    //等待一段时间后去取消协程
    job.cancel() //类似 Thread#interrupt 的作用
}
```

存在阻塞状态（比如 delay、emit 和 withContext 等这类可取消的挂起函数的调用）的情况
```kotlin
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        //模拟协程执行任务
        for (i in 0 until 100000) {
			//调用 job.cancel 会引发 delay 这类挂起函数抛出 JobCancellationException 异常（继承自 CancellationException）从而触发协程的取消
            delay(2) //如果此时要用 try-catch 对异常进行捕获的话，那将会导致协程取消失败，不过可以通过捕获 CancellationException 异常去处理一些清理工作后再抛出
            println("协程运行中 i=$i")
        }
        //
        println("---- 协程结束 ----");
    }
    //演示逻辑
    delay(10)
    //等待一段时间后去取消协程
    job.cancel()
}
```


## ensureActive 扩展函数
- ensureActive 函数就是对判断检查活跃状态并主动抛出一个 CancellationException 异常这一逻辑进行的一个简单封装，所以可以作为定期检查代码中的第一个函数
```kotlin
//扩展函数
public fun CoroutineScope.ensureActive(): Unit = coroutineContext.ensureActive()
//
public fun CoroutineContext.ensureActive() {
    get(Job)?.ensureActive()
}
//
public fun Job.ensureActive(): Unit {
    if (!isActive) throw getCancellationException()
}
//
@InternalCoroutinesApi
public fun getCancellationException(): CancellationException
```

## yield 函数
- yield 函数是一个挂起函数，内部的第一个操作就是调用 ensureActive 函数
- yield 函数比 ensureActive 函数的逻辑要更加复杂，尝试让出协程执行权给调度器，使得其他协程有机会运行，这样可以避免当前协程对 CPU 占用过于密集而导致其它协程一直无法执行的情况
- yield 并不保证立即切换到另一个协程，只是表明当前协程愿意让出执行权，最终还是由协程调度器做出决策


## 总结
- 协程 Job 的 cancel 函数并不会立即中断后续代码的执行，只是将协程的活跃状态 isActive 置为 false，如果协程自身没有正确处理这个状态的话，协程代码会继续运行，直到正常结束
- 协程需要周期性的检查自己的 isActive 活跃状态，然后在代码中根据实际业务情况决定是否继续执行，并在适当的时候停止，这种协作式的设计允许协程在取消时可以进行一些清理工作（比如关闭资源、保存状态等），这也决定了在通常情况下开发者要始终确保代码取消的整体逻辑是协作式的，这种理念也算是对协程内操作数据的一种保护措施（试想一下，代码中途直接取消很可能引起临时数据丢失等问题）
- 另外协程是通过抛出一个 CancellationException 这样的特殊异常让系统来完成协程的取消操作的（相比线程来说不用我们自行处理取消操作，只需要抛个异常就可以自动帮我们做到了）
- 协程里可取消的挂起函数（比如 delay）内部都会进行协程的活跃状态检查，当 Job 的 cancel 调用后会抛出 CancellationException 异常来让系统来取消协程，因此如果代码工作逻辑里已经使用了任意一个可取消的挂起函数，那么额外的判断检查活跃状态状态并主动抛出一个 CancellationException 异常停止协程或者 ensureActive 这样的逻辑就不需要写了







 




 

