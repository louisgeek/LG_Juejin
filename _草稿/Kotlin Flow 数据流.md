# Kotlin Flow 数据流  https://juejin.cn/post/7375525093266292777
- 属于 kotlinx.coroutines.flow 包下
- Flow、SharedFlow 和 StateFlow
- Flow 的设计灵感也来源于响应式流以及其各种实现。但是 Flow 的主要目标是拥有尽可能简单的设计，以及对 kotlin 协程更友好的支持
- 但从概念上讲，Flow 依然是响应式流，和 RxJava 类似，依然有冷热流之分，相比于 RxJava 的切换线程，Flow 也会更加简单
- 流使用 emit 函数发射值，使用 collect 函数收集值
- 通过协程的挂起和恢复机制，Flow 实现了数据的异步传递和处理，Flow 就是利用了协程的这一特性来实现数据流的处理
- 在冷流中，订阅者之间不会共享数据，每个订阅者都会从头开始接收数据，而在热流中，无论有没有订阅者订阅，事件始终都会发生，所有已订阅的订阅者会立即接收到最新的数据。、、、

## Flow
- Flow 是一个接口，有一个 collect 的挂起函数，传递一个 FlowCollector 接口
- 冷流（冷数据流）：当数据被订阅的时候，发布者才开始执行发射数据流的代码，每一次流的收集都是已知的数据，collect 收集完以后就会立即销毁，不能在后续的使用中，如果后续需要发射新的值到该流中，无法再次收集，只能再调用一次 collect 进行收集

```kotlin
public interface Flow<out T> {

    /**
     * Accepts the given [collector] and [emits][FlowCollector.emit] values into it.
     *
     * This method can be used along with SAM-conversion of [FlowCollector]:
     * ```
     * myFlow.collect { value -> println("Collected $value") }
     * ```
     *
     * ### Method inheritance
     *
     * To ensure the context preservation property, it is not recommended implementing this method directly.
     * Instead, [AbstractFlow] can be used as the base type to properly ensure flow's properties.
     *
     * All default flow implementations ensure context preservation and exception transparency properties on a best-effort basis
     * and throw [IllegalStateException] if a violation was detected.
     */
    public suspend fun collect(collector: FlowCollector<T>)
}
```

## FlowCollector
- FlowCollector 是一个接口，有一个 emit 的挂起函数

```kotlin
/**
 * [FlowCollector] is used as an intermediate or a terminal collector of the flow and represents
 * an entity that accepts values emitted by the [Flow].
 *
 * This interface should usually not be implemented directly, but rather used as a receiver in a [flow] builder when implementing a custom operator,
 * or with SAM-conversion.
 * Implementations of this interface are not thread-safe.
 *
 * Example of usage:
 *
 * ```
 * val flow = getMyEvents()
 * try {
 *     flow.collect { value ->
 *         println("Received $value")
 *     }
 *     println("My events are consumed successfully")
 * } catch (e: Throwable) {
 *     println("Exception from the flow: $e")
 * }
 * ```
 */
public fun interface FlowCollector<in T> {

    /**
     * Collects the value emitted by the upstream.
     * This method is not thread-safe and should not be invoked concurrently.
     */
    public suspend fun emit(value: T)
}
```

## flow
- flow 方法会返回一个 SafeFlow 类的对象，SafeFlow 类继承 AbstractFlow 抽象类，AbstractFlow 实现了 Flow, CancellableFlow 两个接口
```kotlin
public fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)
```

### asFlow
- asFlow 是扩展函数，里面调用 flow 函数创建 Flow 的


### flowOf
- flowOf 里面也是调用 flow 函数来创建 Flow 的



## 线程调度
- flowOn 是 Flow 的扩展函数
flowOn 可以将执行此流的上下文更改为指定的上下文
flowOn 可以进行组合使用
flowOn 只影响前面没有自己上下文的操作符，已经有上下文的操作符不受后面 flowOn 的影响
不管 flowOn 如何切换线程，collect 始终是运行在调用它的协程调度器上
 

## Flow 操作符

### 流程操作符（过渡操作符）
- onStart 在上游流启动之前被调用
- onEach 在上游流的每个值被下游发出之前调用
- onCompletion 在流程完成或取消后调用，并将取消异常或失败作为操作的原因参数传递

### 异常操作符
- catch 
 
### 中间运算符（转换操作符）
- transform
- map
- fliter
- zip

### 限制操作符（限长操作符）
- take 
- drop
 
### 终端运算符（末端操作符）
- collect  
- toList



## 常用示例

```
// 从网络请求获取用户列表的函数
suspend fun fetchUsers(): List<User> {
    // ... 发起网络请求并获取数据
}

// 保存用户列表到 Room 数据库的函数
suspend fun saveUsersToDatabase(users: List<User>) {
    // ... 将数据保存到数据库
}

// 在 ViewModel 中使用 Kotlin Flow
class UserViewModel : ViewModel() {
    val usersFlow: Flow<List<User>> = flow {
        try {
            val users = fetchUsers() // 从网络获取用户列表
            saveUsersToDatabase(users) // 保存到数据库
            emit(users) // 发射数据
        } catch (e: Exception) {
            // 处理异常，例如发射一个空列表或错误信息
            emit(emptyList())
            // 或者使用错误状态流
            // errorFlow.emit(e)
        }
    }.flowOn(Dispatchers.IO)
}
```

 


## stateIn
- Flow 的扩展函数，将 Flow 冷数据流变为 StateFlow 热数据流 

## shareIn
- Flow 的扩展函数，将 Flow 冷数据流变为 SharedFlow 热数据流


## StateFlow 和 LiveData

- StateFlow 和 LiveData 有很多相似之处，两者都是可观察的数据容器类，并且在应用架构中使用时，两者都遵循相似模式
- 但 StateFlow 和 LiveData 的行为确实有所不同：StateFlow 需要将初始状态传递给构造函数，而 LiveData 不需要；当 View 进入 STOPPED 状态时，LiveData.observe() 会自动取消注册使用方，而从 StateFlow 或任何其他数据流收集数据的操作并不会自动停止。若要实现相同的行为，您需要从 Lifecycle.repeatOnLifecycle 块收集数据流。

