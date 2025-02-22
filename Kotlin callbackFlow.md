# Kotlin callbackFlow
- callbackFlow 可以将普通的 Callback 回调转换成 Flow 数据流，特别是文本搜索框监听、网络状态监听和传感器数据刷新等场景
- ChannelFlow 是一种特殊的 Flow（实现了 Flow 接口），使用 Channel 来生成和发送数据，结合了 Flow 和 Channel 的特点（缓冲区、跨协程通信等），虽然 ChannelFlow 底层是基于 Channel 实现的（将 Channel 的生产者逻辑封装为 Flow），但在使用上表现为冷流（只有被收集时才开始工作）
- awaitClose 方法是必须配合使用的，可以保证资源被及时释放，避免内存泄漏
- 和 suspendCancelableCoroutine 应用场景有些相似，说直白点 callbackFlow 更适用于多次回调的场景，而 suspendCancelableCoroutine 更适用于单次回调

将回调转换成数据流
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
```

```kotlin
fun fetchDataByCallbackFlow(): Flow<String> = callbackFlow {
    val callback = object : Callback {
        override fun onSuccess(data: String) {
            //发送数据到 Flow 中（trySend 尝试往 Channel 中添加元素，如果 Channel 已经满了，那么就立刻会返回失败）
            val result =  trySend(data)
            //trySendBlocking 尝试往 Channel 中添加元素,当 Channel 已满的时候，会阻塞等待直到有空闲以后添加再返回成功或者 Channel 关闭
            //val result =  trySendBlocking(data)
            val isSuccess = result.isSuccess
            println("trySend: isSuccess=${isSuccess}")
        }

        override fun onFailure(e: Exception) {
            //出现异常时关闭 Channel
            val result = close(e)
            println("close: result=${result}")
        }
    }
    //注册
    fetchData(callback)
    //awaitClose 是必须的，否则会报异常
    awaitClose {
        //流的收集被取消时、协程被 cancel 取消时或显式调用 Channel#close 方法时 awaitClose 才会被调用
        //可以取消注册回调
        println("awaitClose close")
    }
}
//使用
launch {
    try {
        fetchDataByCallbackFlow()
            .collect { data ->
                println("fetchDataByCallbackFlow: data=$data")
            }
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

也可以这么用
```kotlin
interface CallbackWithComplete {
    fun onSuccess(data: String)
    fun onFailure(e: Exception)
    fun onComplete()
}
fun fetchData(callback: CallbackWithComplete) {
    thread {
        //模拟耗时然后返回数据
        Thread.sleep(3000)
        val value = Random.nextInt()
        if (value % 2 == 0) {
            callback.onSuccess("data_$value")
        } else {
            callback.onFailure(Exception("error_$value"))
        }
        callback.onComplete()
    }
}
```

```kotlin
fun fetchDataByCallbackFlow(): Flow<String> = callbackFlow {
    val callback = object : CallbackWithComplete {
        override fun onSuccess(data: String) {
            //发送数据到 Flow 中（trySend 尝试往 Channel 中添加元素，如果 Channel 已经满了，那么就立刻会返回失败）
            val result =  trySend(data)
            //trySendBlocking 尝试往 Channel 中添加元素,当 Channel 已满的时候，会阻塞等待直到有空闲以后添加再返回成功或者 Channel 关闭
//            val result =  trySendBlocking(data)
            val isSuccess = result.isSuccess
            println("trySend: isSuccess=${isSuccess}")
        }

        override fun onFailure(e: Exception) {
            //取消协程
            cancel("callback error", e)
        }

        override fun onComplete() {
            //完成后关闭 Channel
            val result = close()
            println("close: result=${result}")
        }
    }
    //注册
    fetchData(callback)
    //awaitClose 是必须的，否则会报异常
    awaitClose {
        //流的收集被取消时、协程被 cancel 取消时或显式调用 Channel#close 方法时 awaitClose 才会被调用
        //可以取消注册回调
        println("awaitClose close")
    }
}
```


常见使用场景
```kotlin
//比如 EditText 输入框的搜索逻辑
fun TextView.textWatcherFlow(): Flow<String> {
    return callbackFlow {
        val textWatcher = object : TextWatcher {
            override fun afterTextChanged(s: Editable?) {
                s?.let {
                    trySend(s.toString())
                }
            }
            override fun beforeTextChanged(
                s: CharSequence?,
                start: Int,
                count: Int,
                after: Int
            ) { 
            
            }
            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) { 

            }
        }
        //TextView#addTextChangedListener
        addTextChangedListener(textWatcher)
        //awaitClose 方法是必须的，用于处理资源的关闭
        awaitClose {
            //通常在这里移除取消回调注册
            //TextView#removeTextChangedListener
            removeTextChangedListener(textWatcher)
        }
    }
}
```



