# Kotlin 知识点

## measureTimeMillis 和 measureNanoTime
```kotlin
//currentTimeMillis
val costTime = measureTimeMillis { 
    //需要统计耗时的代码
}
//nanoTime
val costTime = measureNanoTime { 
    //需要统计耗时的代码
}
```

```kotlin
public inline fun measureTimeMillis(block: () -> Unit): Long {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val start = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - start
}

public inline fun measureNanoTime(block: () -> Unit): Long {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val start = System.nanoTime()
    block()
    return System.nanoTime() - start
}
```


## kotlin.repeat
```kotlin
repeat(5) {
    println("test $it")
}
//--------------
test 0
test 1
test 2
test 3
test 4
```

```kotlin
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }
    //就是 for 循环
    for (index in 0 until times) {
        action(index)
    }
}
```




## kotlin.runCatching 和 T.runCatching
```kotlin
kotlin.runCatching {
    //需要 try-catch 的代码
    "data" //返回值
}.onSuccess { data ->
}.onFailure { throwable ->
}

obj.runCatching {
    //直接访问 obj 属性
    //需要 try-catch 的代码
    "data" //返回值
}.onSuccess { data ->
}.onFailure { throwable ->
}
```

```kotlin
@InlineOnly
@SinceKotlin("1.3")
public inline fun <R> runCatching(block: () -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}

@InlineOnly
@SinceKotlin("1.3")
public inline fun <T, R> T.runCatching(block: T.() -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}
```



# Kotlin Coroutines 协程知识点

## Coroutines 协程是什么
- 广义上的协程：和线程类似，也是一种解决异步任务的方案
- Kotlin 的协程：可以理解是对线程的一种封装，一种可以让异步任务的代码写成一种类似同步的形式，可以看作是一个线程封装框架，官方称为一种轻量级线程
    - 可以解决异步任务代码的嵌套
    - 必须依赖线程存在，但是可以在不同的线程之间切换
    - 由开发者管理，不需要操作系统进行调度和切换

## CoroutineContext 协程上下文
- CoroutineContext 接口的设计和 Map 很相识，也是根据 Key 取值的，相同 Key 的值会被覆盖，支持合并（CoroutineContext 重载了 plus 操作符，所以可以用 + 号组合），可以看作是一系列 CoroutineContext.Element 元素的集合
    - CoroutineName 代表协程的名称，通常可用于调试时候的识别
    - Job 用于管理协程生命周期，算是已启动协程的句柄，比如可以调用 job.cancel() 取消正在运行的协程
    - CoroutineDispatcher（Dispatchers）用于协程处理线程调度，将任务分派到当的线程
    - CoroutineExceptionHandler 用于协程的异常处理，处理未捕获的异常
    - ContinuationInterceptor 的作用是拦截协程的，CoroutineDispatcher 也实现了 ContinuationInterceptor 接口

## suspend 挂起函数
- CPS 转换：Kotlin 编译器会将 suspend 挂起函数转换成一个带有 Callback 参数的回调函数（为其额外添加 Continuation 类型的参数），这里的 Callback 回调接口就是 Continuation 接口
- 简单的理解就是协程使用 Continutation 替换传统的 Callback，每一个协程程序的创建都会伴随 Continutation 的存在，同时协程创建的时候都会自动回调一次 Continutation 的 resumeWith 方法，以便让协程开始执行