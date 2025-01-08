# Kotlin suspendCoroutine 和 suspendCancellableCoroutine 的区别
- 两个函数的作用都是允许在协程中将回调式的异步非挂起的操作转化为挂起操作，换句话说就是将异步操作封装为挂起操作，用于适配已有的回调式代码，是连接异步回调函数和协程之间的桥梁
- 将回调函数转换为协程挂起函数，比如 onSuccess 走 Continuation.resume, 而 onFailure 走 Continuation.resumeWithException
- suspendCancellableCoroutine 的基本功能和 suspendCoroutine 一样，只是多了一个监听协程取消的功能，更推荐使用

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}
@InlineOnly
public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

@InlineOnly
public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))
```

```kotlin
//suspendCoroutine 函数的参数是一个 Continuation 接口
public suspend inline fun <T> suspendCoroutine(crossinline block: (Continuation<T>) -> Unit): T {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    return suspendCoroutineUninterceptedOrReturn { c: Continuation<T> ->
        val safe = SafeContinuation(c.intercepted())
        block(safe)
        safe.getOrThrow()
    }
}
//suspendCancellableCoroutine 函数的参数是一个 CancellableContinuation 接口（继承自 Continuation），支持取消
public suspend inline fun <T> suspendCancellableCoroutine(
    crossinline block: (CancellableContinuation<T>) -> Unit
): T =
    suspendCoroutineUninterceptedOrReturn { uCont ->
        val cancellable = CancellableContinuationImpl(uCont.intercepted(), resumeMode = MODE_CANCELLABLE)
        /*
         * For non-atomic cancellation we setup parent-child relationship immediately
         * in case when `block` blocks the current thread (e.g. Rx2 with trampoline scheduler), but
         * properly supports cancellation.
         */
        cancellable.initCancellability()
        block(cancellable)
        cancellable.getResult()
    }
```

## suspendCoroutine
- 将普通函数转化成挂起函数

```kotlin
interface Callback {
    fun onSuccess(data: String)
    fun onFailure(e: Exception)
}
fun fetchData(callback: Callback) {
    thread {
        //模拟耗时然后返回数据
        Thread.sleep(3000)
        val value = Random.nextInt()
        if (value % 2 == 0) {
            callback.onSuccess("data_$value")
        } else {
            callback.onFailure(Exception("error_$value"))
        }
    }
}
//使用
val callback = object : Callback {
    override fun onSuccess(data: String) {
        println("onSuccess: data=$data")
    }
    override fun onFailure(e: Exception) {
        println("onFailure: msg=${e.message}")
    }
}
fetchData(callback) //设置回调
```

```kotlin
//挂起函数
suspend fun fetchDataBySuspendCoroutine(): String = suspendCoroutine { continuation ->
    val callback = object : Callback {
        override fun onSuccess(data: String) {
            continuation.resume(data)
        }
        override fun onFailure(e: Exception) {
            continuation.resumeWithException(e)
        }
    }
    fetchData(callback)
}
//使用
launch {
    //要么返回 data 然后打印，要么抛出异常被 try-catch 捕获
    try {
        val data = fetchDataBySuspendCoroutine()
        println("fetchDataBySuspendCoroutine: data=$data")
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

常见使用场景

```kotlin
//创建一个 View 的扩展函数 await
suspend fun View.await() = suspendCoroutine { continuation ->
    this.post {
        //在 View.post 函数中将 View 本身返回
        continuation.resume(this)
    }
}
//使用
launch {
    val _view = view.await()
    val width = _view.width
    val height = _view.height
    Log.e("TAG", "width=$width height=$height")
}
```

## suspendCancellableCoroutine
- 将普通函数转化成可取消的挂起函数

```kotlin
suspend fun fetchDataBySuspendCancellableCoroutine(): String =
    suspendCancellableCoroutine { continuation ->
        val callback = object : Callback {
            override fun onSuccess(data: String) {
                //
                if (continuation.isCancelled) {
                    //检测协程是否被取消，如果已经取消则直接返回不继续执行后续操作
                    return
                }
                continuation.resume(data)
            }

            override fun onFailure(e: Exception) {
                continuation.resumeWithException(e)
            }
        }
        fetchData(callback)
    }
```

利用 invokeOnCancellation 函数
- invokeOnCancellation 会在协程被取消的时候会被调用，可以执行清理操作、释放资源等操作
```kotlin
suspend fun awaitCallback(): String = suspendCancellableCoroutine { continuation ->
    val api = Api()
    //Callback 回调
    val callback = object : Callback {
        override fun onSuccess(data: String) {
            continuation.resume(data)
        }
        override fun onFailure(e: Exception) {
            continuation.resumeWithException(e)
        }
    }
    api.addCallback(callback)
    //
    continuation.invokeOnCancellation { throwable ->
        //可以移除 Callback 回调
        api.removeCallback(callback)
    }
}
```

常见使用场景
- retrofit2.Call 的 awaitResponse 扩展函数也是使用 suspendCancellableCoroutine 函数进行转换的
```kotlin
//Retrofit 能够支持协程的原理
suspend fun <T> Call<T>.awaitResponse(): Response<T> {
  return suspendCancellableCoroutine { continuation ->
    //协程被取消时被调用
    continuation.invokeOnCancellation { throwable ->
      //就是 Call#cancel
      cancel()
    }
    //就是 Call#enqueue
    enqueue(object : Callback<T> {
      override fun onResponse(call: Call<T>, response: Response<T>) {
        continuation.resume(response)
      }

      override fun onFailure(call: Call<T>, t: Throwable) {
        continuation.resumeWithException(t)
      }
    })
  }
}
```
